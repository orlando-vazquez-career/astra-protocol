# Mapa del repositorio ASTRA

Qué hay en cada carpeta y para qué sirve. Si un archivo no está aquí, o falta en el mapa o sobra en el repo.

```
ASTRA/
├─ ASTRA-PROTOCOL.md               documento maestro: fases, gates, axiomas, roles, multi-cadena, multi-vendor, herramientas, version history
├─ README.md                       presentación y cómo usarlo en un proyecto
├─ AGENTS.md                       reglas para agentes que trabajan en ESTE repo (fuente única)
├─ CLAUDE.md                       Claude Code → importa @AGENTS.md
├─ GEMINI.md                       Gemini CLI / Antigravity → remite a AGENTS.md
├─ LICENSE                         MIT, DevZen SpA 2026
├─ .gitignore · .gitattributes     higiene (sin .env) y finales de línea LF
│
├─ guides/                         el CÓMO de cada parte del protocolo
│  ├─ README.md                    índice y cuándo leer cada guía
│  ├─ gates.md                     Gate 1 (Carta) y Gate 2 (Mainnet): checklists, cómo se registran, qué pasa si fallan
│  ├─ keys-and-secrets.md          A2 y A7 operativos: alias por cadena, incidentes, qué detecta astra check
│  ├─ audit-checklist.md           checklist universal + Stellar + EVM + L2 + Syscoin; formato de audit.md
│  ├─ standards-map.md             caso de uso → estándar por familia (A5)
│  ├─ agentic-payments.md          x402 y MPP: cuándo cada uno, qué va en la Carta, qué mira la Auditoría
│  ├─ runtimes.md                  tiers y configuración por runtime de IA; pruebas de humo
│  ├─ skills-externas.md           política y mapa del pack de skills Stellar; lo que NO es ASTRA
│  └─ chains/
│     ├─ stellar.md                Soroban: redes, CLI, direcciones, ciclo, verificación, costos, riesgos
│     ├─ syscoin.md                L1 UTXO, NEVM (57/5700) y Rollux (570/57000)
│     ├─ base.md                   Base (8453) y Base Sepolia (84532)
│     └─ evm-generico.md           cualquier EVM y cómo agregarla al registro
│
├─ templates/                      plantillas de artefactos y roster de agentes
│  ├─ orbit.template.md            Fase 1
│  ├─ chart.template.md            Fase 2 · Gate 1 (línea aprobada:)
│  ├─ audit.template.md            Fase 5 (veredicto: y tabla de hallazgos)
│  ├─ launch.template.md           Gate 2 · Fase 6 (firmado_por, fecha_firma, costo_estimado, alias_mainnet, plan_rollback)
│  ├─ devlog.template.md           Fase 7
│  ├─ deployments.schema.json      JSON Schema de .astra/deployments.json
│  └─ agents/                      navegante · cartografo · forjador · auditor-de-cadena · oficial-de-lanzamiento (+ README)
│
├─ skills/                         skills canónicas (formato Agent Skills)
│  ├─ astra/                       punto de entrada: ¿en qué fase estamos?
│  ├─ astra-orbit/ · astra-chart/ · astra-build/ · astra-testnet/ · astra-audit/ · astra-launch/ · astra-logbook/
│  └─ astra-standards/             estándar antes que invento
├─ .claude/skills/                 copia GENERADA para Claude Code (astra skills sync)
├─ .agents/skills/                 copia GENERADA para Codex y Antigravity
│
├─ docs/
│  ├─ GLOSSARY.md                  vocabulario del protocolo
│  ├─ MAPA.md                      este archivo
│  └─ superpowers/
│     ├─ specs/2026-09-04-astra-protocol-design.md      diseño de la génesis
│     └─ plans/2026-09-04-astra-genesis-plan.md         plan de implementación de la génesis
│
└─ genesis/                        historia fundacional (no se reescribe)
   ├─ README.md
   └─ devlogs/2026-09-04-genesis.md
```

La herramienta `astra` vive en el repo hermano `astra-cli` (`bin/`, `lib/`, `data/chains.json`, `data/standards.json`, `test/`).
