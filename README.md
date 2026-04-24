# OpenCode Configuration

Configuración personalizada de [OpenCode](https://opencode.ai) con workflow **Spec-Driven Development (SDD)** y agente mentor "Tony Stark".

## Estructura

```
.config/opencode/
├── AGENTS.md           # Personalidad y reglas del agente "tony stark"
├── opencode.json       # Configuración principal de agentes y MCPs
├── package.json        # Dependencias del plugin
├── commands/           # Comandos rápidos SDD
├── plugins/            # Plugins TypeScript personalizados
├── prompts/            # Prompts de los agentes SDD
└── skills/             # Skills reutilizables por contexto
```

## Agente: Tony Stark

**Rol:** Senior Architect (15+ años de experiencia, GDE & MVP)

**Filosofía:**
- CONCEPTOS > CÓDIGO
- La IA es una herramienta; el humano siempre manda
- Fundamentos SOLID, arquitectura limpia/hexagonal
- Contra la inmediatez: sin atajos, el aprendizaje real requiere esfuerzo

**Comportamiento:**
- Profesor apasionado que quiere que crezcas
- Te corrige con evidencia técnica cuando estás errado
- Usa analogías de construcción/arquitectura para explicar conceptos
- Habla con modismos de la costa colombiana

## Workflow SDD (Spec-Driven Development)

Sistema de agentes coordinados para cambios estructurados:

| Agente | Función |
|--------|---------|
| `sdd-init` | Inicializar contexto SDD en un proyecto |
| `sdd-explore` | Explorar codebase e investigar ideas |
| `sdd-propose` | Crear propuestas de cambio |
| `sdd-design` | Diseño técnico con decisiones de arquitectura |
| `sdd-spec` | Especificaciones detalladas con escenarios |
| `sdd-tasks` | Desglose en tareas de implementación |
| `sdd-apply` | Ejecutar cambios desde las tareas |
| `sdd-verify` | Validar implementación vs especificaciones |
| `sdd-archive` | Archivar artefactos completados |
| `sdd-onboard` | Guía completa del ciclo SDD |

**Comandos disponibles:**
- `/sdd-new` - Crear nueva change
- `/sdd-ff` - Fast-forward: aplicar cambios directamente
- `/sdd-continue` - Continuar workflow interrumpido

## MCP Servers

| Servidor | Tipo | Propósito |
|----------|------|-----------|
| `context7` | Remoto | Documentación actualizada de librerías |
| `engram` | Local | Memoria persistente entre sesiones |
| `chrome-devtools` | Local | Integración con DevTools |

## Skills (Auto-load por contexto)

Se activan automáticamente según el contexto detectado:

| Contexto | Skill |
|----------|-------|
| Go tests, Bubbletea TUI | `go-testing` |
| Crear nuevas AI skills | `skill-creator` |
| Workflow SDD | `sdd-*` |
| Crear PRs | `branch-pr` |
| Crear issues | `issue-creation` |
| Review adversarial | `judgment-day` |

## Plugins Personalizados

- `background-agents.ts` - Agentes en segundo plano
- `engram.ts` - Integración con memoria persistente

## Uso

### Inicializar SDD en un proyecto
```
/sdd-init
```

### Crear un cambio nuevo
```
/sdd-new
```

### Workflow completo guiado
```
/sdd-onboard
```

### Activar agente Tony Stark
```
/tony stark
```

## Dependencias

```json
{
  "@opencode-ai/plugin": "^1.4.6",
  "unique-names-generator": "^4.7.1"
}
```

---

**Nota:** Esta configuración está diseñada para desarrollo profesional con énfasis en arquitectura, testing y aprendizaje continuo.
