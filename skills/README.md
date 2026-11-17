# Skills del repositorio

Skills operativas propiedad de **TAMV ONLINE NETWORK** para este proyecto. No sustituyen
`AGENTS.md`: lo implementan en tareas concretas.

| Skill                       | Ubicación                                   | Propósito                                                        | Herramientas            |
| --------------------------- | ------------------------------------------- | ---------------------------------------------------------------- | ----------------------- |
| `tamv-browser-verification` | `skills/tamv-browser-verification/SKILL.md` | Verificación web, QA y evidencia observable con `agent-browser`. | `Bash(agent-browser:*)` |

## Activación

Este repositorio usa `skills/` como ubicación canónica. Si tu cliente de agentes carga skills
desde otro directorio, copia o enlaza la carpeta `skills/tamv-browser-verification/` a ese
directorio; no dupliques ni edites el contenido en dos lugares.

## Reglas

- Una skill debe existir, estar versionada y poder verificarse. No declares skills inexistentes.
- Toda skill nueva documenta: propósito, alcance, herramientas permitidas, límites de seguridad,
  evidencia que produce y condiciones de parada.
- Las skills no conceden autoridad: no pueden aprobar producción, mover dinero ni alterar datos
  protegidos.
- La evidencia generada se guarda en rutas ignoradas por Git (por ejemplo `.output/qa/`).
- Antes de instalar una skill de terceros, revisa qué comandos ejecuta y qué datos puede leer.
