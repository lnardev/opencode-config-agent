---
description: Revisión de PR pasando la URL de Bitbucket directamente — /pr-review <url_del_pr>
---

# PR Review — Bitbucket Cloud

Uso: `/pr-review https://bitbucket.org/workspace/repo/pull-requests/123`

Ejemplo:
```
/pr-review https://bitbucket.org/grupokonecta/teo_rest_v2/pull-requests/1314
```

---

## 1. Configuración requerida

Solo necesitas el token — workspace y repo se extraen de la URL automáticamente:

```bash
export BITBUCKET_TOKEN="token_example"
export BITBUCKET_USER="user@example.com"
```

Token en: **Bitbucket → Personal settings → API tokens → Create API token**
Permisos mínimos: `Repositories: Read` + `Pull requests: Read`

Si no se proporciona URL → responde: **"Uso: /pr-review <url_del_pr>"** y termina.

---

## 2. Parsear la URL y extraer parámetros

Dado `$ARGUMENTS` con formato `https://bitbucket.org/{workspace}/{repo}/pull-requests/{pr_id}`, extrae:

```bash
URL="$ARGUMENTS"

# Extraer componentes de la URL
BITBUCKET_WORKSPACE=$(echo "$URL" | awk -F'/' '{print $4}')
BITBUCKET_REPO=$(echo "$URL" | awk -F'/' '{print $5}')
PR=$(echo "$URL" | awk -F'/' '{print $7}')

echo "Workspace: $BITBUCKET_WORKSPACE"
echo "Repo:      $BITBUCKET_REPO"
echo "PR:        $PR"
```

Si alguno de los tres valores queda vacío → responde: **"URL inválida. Formato esperado: https://bitbucket.org/workspace/repo/pull-requests/123"** y termina.

---

## 3. Obtener datos del PR desde Bitbucket Cloud API

```bash
AUTH="-u $BITBUCKET_USER:$BITBUCKET_TOKEN"
BASE="https://api.bitbucket.org/2.0/repositories/$BITBUCKET_WORKSPACE/$BITBUCKET_REPO"

# 1. Metadata del PR
curl -s $AUTH "$BASE/pullrequests/$PR" \
  | jq '{
      id,
      title,
      description,
      author: .author.display_name,
      source: .source.branch.name,
      destination: .destination.branch.name,
      state,
      created_on,
      updated_on
    }'

# 2. Diff completo (código fuente de los cambios)
curl -s $AUTH "$BASE/pullrequests/$PR/diff"

# 3. Archivos modificados
curl -s $AUTH "$BASE/pullrequests/$PR/diffstat" \
  | jq '.values[] | {status, path: .new.path}'

# 4. Commits del PR
curl -s $AUTH "$BASE/pullrequests/$PR/commits" \
  | jq '.values[] | {hash: .hash[0:8], message, author: .author.raw}'

# 5. Comentarios existentes (para no duplicar hallazgos ya reportados)
curl -s $AUTH "$BASE/pullrequests/$PR/comments" \
  | jq '.values[] | {author: .author.display_name, content: .content.raw}'
```

**Manejo de errores:**
- **401** → token inválido o expirado
- **403** → sin permisos sobre el repo
- **404** → PR no existe o URL incorrecta

---

## 4. Análisis SonarQube sobre el diff del PR

Analiza **únicamente las líneas añadidas o modificadas** en el diff. Usa el resto como contexto.

### 🐛 Bugs
- Variables/propiedades indefinidas, null dereference
- TypeError, casting incorrecto
- Catch de excepción de clase que no existe en el namespace (falta `\` o `use`)
- Condiciones siempre true/false, código inalcanzable
- Catches vacíos o que silencian excepciones
- Off-by-one, `==` en lugar de `===`
- Recursos no cerrados (DB, file handles, streams)
- División por cero sin protección
- Retorno implícito null en función con tipo declarado
- Uso de variable antes de asignación en todas las ramas posibles

### 🔒 Vulnerabilities
- **Injection**: SQL, NoSQL, LDAP, OS command, XPath, Code (`eval`, `create_function`, `preg_replace /e`)
- **XSS** (reflected, stored, DOM), **CSRF**, **SSRF**, **XXE**
- **Path traversal**, file upload sin validación
- `unserialize()` con input externo
- **Criptografía débil**: MD5/SHA1 para passwords, DES, ECB, IV estático
- **Hardcoded secrets**: API keys, passwords, tokens, connection strings
- Rutas sin middleware de autenticación/autorización
- Cookies sin `HttpOnly`/`Secure`/`SameSite`
- `CURLOPT_SSL_VERIFYPEER = false` o TLS deshabilitado
- Input sin validación antes de operaciones críticas
- Ausencia de logging en operaciones críticas (auth, pagos, cambios de permisos)

### 🔥 Security Hotspots
- `rand()`/`mt_rand()` en contextos de seguridad
- `unserialize()` aunque el input parezca controlado
- Regex con riesgo de ReDoS
- CORS permisivo (`*`)
- `eval()`, `exec()`, `system()`, `passthru()`
- `phpinfo()` en cualquier archivo
- `error_reporting(E_ALL)` o `display_errors = On`
- HTTP en lugar de HTTPS en URLs externas
- Logging de datos sensibles o PII
- `chmod 777` o permisos de archivo amplios
- `md5()`/`sha1()` para hashing de contraseñas

### 👃 Code Smells
- Cognitive Complexity alta
- Métodos > 50 líneas, clases > 300 líneas, anidación > 4 niveles
- Parámetros > 7 en una función
- Duplicación de lógica (bloques ≥ 10 líneas repetidos)
- Dead code, imports/variables no usados
- Magic numbers, strings hardcodeados
- Violaciones SOLID, acoplamiento excesivo
- `switch` sin `default`, catches vacíos
- Funciones deprecadas (`mysql_*`, `ereg_*`, `split()`)
- `dd()`, `var_dump()`, `print_r()`, `die()`, `console.log()` olvidados
- Código comentado en lugar de borrado
- `TODO`/`FIXME` sin ticket asociado
- Métodos vacíos sin justificación

---

## 5. Checklist de PR

- [ ] La lógica resuelve lo declarado en título/descripción del PR
- [ ] Sin regresiones en funcionalidad existente
- [ ] Tests cubren nueva funcionalidad y casos de error
- [ ] `.env.example` actualizado si hay nuevas variables
- [ ] Breaking changes declarados en la descripción
- [ ] Migraciones de BD documentadas

---

## 6. Formato de salida

```
## PR #<N> — <título>
🔗 <url_original>
Autor: <autor> | <source> → <destination>
Estado: <state>
Descripción: <resumen>
Commits: N | Archivos modificados: N

---

## Análisis SonarQube

### 🔴 Bloqueantes (corregir antes de mergear)

[SEVERIDAD] [CATEGORÍA] archivo:línea
  Problema: <descripción>
  Impacto:  <por qué importa>
  Fix:
    <código corregido>

### 🟡 Sugerencias (no bloquean)
[archivo:línea] <descripción>

### ✅ Estado por categoría
- Bugs:              ✅ Ninguno / ❌ N hallazgos
- Vulnerabilities:   ✅ Ninguno / ❌ N hallazgos
- Security Hotspots: ✅ Ninguno / ❌ N hallazgos
- Code Smells:       ✅ Ninguno / ❌ N hallazgos

---

## Resumen

| Categoría         | Blocker | High | Medium | Low |
|-------------------|---------|------|--------|-----|
| Bugs              |         |      |        |     |
| Vulnerabilities   |         |      |        |     |
| Security Hotspots |         |      |        |     |
| Code Smells       |         |      |        |     |

---

## Comentarios existentes en Bitbucket
<resumen de lo ya señalado — no duplicar>

---

## Veredicto
[ ] ✅ Aprobar — listo para merge
[ ] 🔄 Solicitar cambios — corregir bloqueantes
[ ] ❌ Rechazar — requiere rediseño

Razón: <justificación>
```

**Severidades:** `BLOCKER` · `HIGH` · `MEDIUM` · `LOW` · `INFO`
**Categorías:** `BUG` · `VULNERABILITY` · `HOTSPOT` · `CODE_SMELL`

---

## 7. Reglas de operación

- Parsear siempre workspace/repo/PR desde la URL — nunca hardcodear
- La fuente de verdad es el diff de Bitbucket — no usar `git diff` local
- Analizar solo líneas añadidas/modificadas; el resto es contexto
- Citar siempre archivo y línea
- Agrupar hallazgos del mismo patrón en un solo item con lista de ubicaciones
- `dd()`, secrets hardcodeados, SQL injection, `phpinfo()` → siempre `BLOCKER`
- Considerar comentarios existentes de Bitbucket para no duplicar
- Si una categoría pasa limpia, confirmarlo explícitamente