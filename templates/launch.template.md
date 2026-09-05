<!--
INSTRUCCIONES — launch.template.md (Gate 2: Mainnet · Fase 6: Lanzamiento)

1. Lo prepara el oficial-de-lanzamiento; lo firma la PERSONA. Un agente jamás completa
   `firmado_por`.
2. `astra check --gate mainnet` exige: firmado_por sin placeholders, fecha_firma en
   YYYY-MM-DD y costo_estimado no vacío, además de carta aprobada, testnet registrada,
   auditoría apta y cero secretos.
3. El alias de mainnet es distinto del de testnet (A2). Aquí va el NOMBRE del alias,
   nunca una clave.
4. Copiar a docs/astra/launch.md (astra init lo hace). Borrar este bloque al completar.
-->

# Lanzamiento — <nombre del artefacto>

**Cadena mainnet**: <id del registro>
**Commit a lanzar**: <sha> (mismo que la entrada testnet <dpl_...> o explicar la diferencia)
firmado_por:
fecha_firma:
costo_estimado:
alias_mainnet:
plan_rollback:

## Checklist del Gate 2

- [ ] `astra check --gate mainnet` en OK (pegar la salida con fecha)
- [ ] Hallazgos críticos y altos cerrados y re-probados en testnet
- [ ] Costo estimado (deploy + inicialización + reservas/rent + prueba de humo) y fondos disponibles en el alias
- [ ] Alias de mainnet distinto del de testnet; la clave nunca pasó por chat, repo ni log
- [ ] Plan de pausa / upgrade / rollback con responsable y tiempos
- [ ] Verificación pública prevista: <explorer, comando, quién confirma>
- [ ] Prueba de humo prevista: <operación, monto mínimo, quién la ejecuta>

## Comandos exactos (los ejecuta la persona)

```
<stellar contract deploy --wasm ... --source-account <alias_mainnet> --network mainnet>
<forge create ... --rpc-url ... --account <alias_mainnet> --broadcast>
```

## Resultado

| Paso | Hecho por | Fecha | Evidencia |
|---|---|---|---|
| Deploy | <persona> | <fecha> | <tx> |
| Registro (`astra deployments add`) | <agente/persona> | <fecha> | <dpl_...> |
| Verificación pública (A6) | <persona> | <fecha> | <url> |
| Prueba de humo | <persona> | <fecha> | <tx> |
| Anuncio | <persona> | <fecha> | <dónde> |

## Intentos

<fecha · motivo del rechazo · a qué fase volvió>
