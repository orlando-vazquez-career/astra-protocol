---
name: astra-logbook
description: Fase 7 de ASTRA (Bitacora). Escribe el devlog del sprint en docs/astra/devlogs/ (direcciones registradas, costos estimados vs reales, hallazgos y cierres, deuda, aprendizajes) y verifica que deployments.json, orbit.md, chart.md y audit.md digan la verdad del estado final. Usar cuando el usuario dice "bitacora", "devlog", "cerrar el sprint", "documentar lo desplegado".
---
<!-- GENERADO por astra skills sync desde skills/astra-logbook/SKILL.md — no editar a mano; editar el canonico y re-correr el sync -->

# astra-logbook — Fase 7: Bitácora

## Cuándo

Al cerrar un sprint (después del Lanzamiento, o al pausar un sprint que no llegó). Produce `docs/astra/devlogs/YYYY-MM-DD-<slug>.md`.

## Pasos

1. Lee `.astra/deployments.json` (`astra deployments list`), `docs/astra/orbit.md`, `chart.md`, `audit.md`, `launch.md` y el historial de commits del sprint.
2. Escribe el devlog desde `templates/devlog.template.md`: cadena, fases recorridas y fechas de gates; qué se hizo; tabla de direcciones registradas (solo las del registro); costos estimados vs reales por operación (A10); auditoría (hallazgos por severidad, cómo se cerraron, cuáles se aceptaron y por quién); lo que quedó fuera y la deuda; aprendizajes.
3. Coherencia: si el devlog dice algo que un artefacto no dice (una dirección no registrada, un cambio de diseño no anotado en la Carta, un hallazgo cerrado que sigue "abierta" en la auditoría), corrige el artefacto, no solo el devlog.
4. `astra check` en OK sobre el commit final. Commit `astra: bitácora <slug>`.

## No hacer

- Sin claves, sin datos personales, sin direcciones que no estén en el registro.
- No embellecer: los costos reales y los hallazgos van como fueron.

## Done

Devlog escrito y ningún artefacto contradice lo desplegado.
