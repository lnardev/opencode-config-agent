---
description: Análisis estilo SonarQube sobre código sin commitear — Bugs, Vulnerabilities, Code Smells, Security Hotspots
---
 
# Code Review estilo SonarQube — Código sin commitear
 
Analiza **únicamente el código que aún no ha sido commiteado** en el working tree actual.
 
## 1. Detectar cambios sin commitear
 
```bash
git diff --cached --name-only   # staged
git diff --name-only             # unstaged
git ls-files --others --exclude-standard  # untracked
git diff HEAD                    # diff completo
```
 
- ✅ Analizar: archivos modificados, agregados o sin trackear
- ❌ NO analizar: código ya commiteado sin cambios pendientes
- ⚠️ Para archivos modificados, enfócate en **las líneas cambiadas**; usa el resto solo como contexto
Si no hay cambios sin commitear → responde **"No hay cambios sin commitear."** y termina.
 
---
 
## 2. Las 4 categorías a revisar
 
### 🐛 Bugs
Código que produce comportamiento inesperado en runtime.
- Variables/propiedades indefinidas, null dereference
- TypeError, casting incorrecto
- Condiciones siempre true/false, código inalcanzable
- Catches vacíos o que silencian excepciones
- Async/await mal usado, promesas no esperadas
- Off-by-one, `==` en lugar de `===`
- Recursos no cerrados (DB, file handles, streams)
- División por cero sin protección
### 🔒 Vulnerabilities
Código explotable por un atacante.
- SQL/NoSQL/OS/LDAP Injection
- XSS (reflected, stored, DOM), CSRF, SSRF, XXE
- Path traversal, file upload sin validación
- Deserialización insegura
- Criptografía débil: MD5, SHA1, DES, ECB, random no seguro
- Hardcoded secrets: API keys, passwords, tokens, connection strings
- Broken authentication/authorization
- Cookies sin `HttpOnly`/`Secure`/`SameSite`
### 🔥 Security Hotspots
Código sensible que requiere revisión humana (no es vulnerabilidad confirmada, pero puede serlo).
- `rand()` / `Math.random()` en contextos de seguridad
- Regex con riesgo de ReDoS
- CORS permisivo (`*`)
- `eval()`, `exec()`, `Function()`
- HTTP en lugar de HTTPS
- Logging de datos sensibles o PII
- Configuración TLS/SSL personalizada
### 👃 Code Smells
Código que funciona pero dificulta el mantenimiento.
- Cognitive Complexity alta (método difícil de entender a primera vista)
- Métodos > 50 líneas, clases > 300 líneas
- Anidación > 4 niveles, parámetros > 7
- Duplicación de lógica (bloques ≥ 10 líneas repetidos)
- Dead code, imports/variables no usados
- Magic numbers, strings hardcodeados
- Violaciones SOLID, acoplamiento excesivo
- `dd()`, `console.log`, `var_dump`, `print_r` olvidados
- Código comentado en lugar de borrado
- `TODO`/`FIXME` sin ticket asociado
---
 
## 3. Formato de salida
 
Por cada hallazgo:
 
```
[SEVERIDAD] [CATEGORÍA] archivo:línea
  Problema: <descripción concisa>
  Impacto:  <por qué importa>
  Fix:
    <código corregido>
```
 
**Severidades:** `BLOCKER` · `HIGH` · `MEDIUM` · `LOW` · `INFO`
**Categorías:** `BUG` · `VULNERABILITY` · `HOTSPOT` · `CODE_SMELL`
 
### Resumen final
 
```
## Archivos revisados: N (staged: N | unstaged: N | untracked: N)
 
| Categoría          | Blocker | High | Medium | Low | Info |
|--------------------|---------|------|--------|-----|------|
| Bugs               |         |      |        |     |      |
| Vulnerabilities    |         |      |        |     |      |
| Security Hotspots  |         |      |        |     |      |
| Code Smells        |         |      |        |     |      |
 
## ¿Listo para commitear?
[ ] ✅ Sí
[ ] ❌ No — corregir antes: <lista de bloqueantes>
```
 
---
## 6. Reglas de operación
- Solo reporta sobre líneas nuevas/modificadas
- Cita siempre archivo y número de línea
- Agrupa hallazgos del mismo patrón repetido en un solo reporte con múltiples ubicaciones
- `dd()`, `console.log`, secrets hardcodeados → siempre `BLOCKER`
- Si los cambios están limpios, dilo explícitamente
- No reportes problemas en código que el usuario no ha tocado