# Gates humanos — Carta (Gate 1) y Mainnet (Gate 2)

Los gates son los dos puntos donde una persona detiene el protocolo, lee y decide. Un agente puede preparar todo lo que hay antes de un gate; no puede cruzarlo.

## Gate 1 — Carta aprobada

**Cuándo**: al terminar la fase Carta, antes de escribir una línea de código.

**Quién**: la persona responsable del proyecto (la que va a firmar el Gate 2 después).

**Qué revisa** en `docs/astra/chart.md`:

- [ ] La interfaz pública es la mínima que resuelve el problema; cada función tiene quién puede llamarla y qué error devuelve si no.
- [ ] Los invariantes están escritos como frases falsables ("el total emitido nunca supera el cap"), no como intenciones.
- [ ] Roles y permisos: quién administra, quién pausa, quién actualiza, cómo se rota cada uno. Si no hay forma de pausar ni de actualizar, dice por qué se acepta.
- [ ] Economía: fees por operación, reservas o rent del storage, límites por transacción; y quién paga cada cosa.
- [ ] Estándares: cada uno con su enlace y con la decisión de implementarlo completo o parcial (A5).
- [ ] Plan de tests: qué se prueba unitario, qué por propiedad o fuzz, qué en testnet.
- [ ] Plan de upgrade / pausa / rollback, o la declaración explícita de que el contrato es inmutable y por qué.
- [ ] Lo que queda fuera está listado.
- [ ] El modelo de amenazas de `orbit.md` sigue siendo válido con este diseño.

**Cómo se registra**: la persona escribe en la cabecera de `chart.md` la línea

```
aprobada: 2026-09-04
```

`astra check --gate mainnet` busca exactamente esa línea (fecha ISO). Sin ella, el Gate 2 nunca abre.

**Si se rechaza**: se anota en `chart.md`, sección "Decisiones del Gate 1", qué cambia y por qué; la Carta vuelve a Fase 2. No se empieza a construir "mientras tanto".

## Gate 2 — Mainnet

**Cuándo**: después de la Auditoría, antes de cualquier operación con dinero real.

**Quién**: la misma persona del Gate 1. Firma con su nombre; un agente jamás completa `firmado_por`.

**Verificación mecánica primero**:

```
astra check --gate mainnet
```

Tiene que mostrar en OK los cinco ítems: `carta-aprobada`, `testnet-desplegada`, `auditoria-apta`, `launch-firmado`, `sin-secretos`. El comando sale con código 1 si alguno falla; ese 1 es un veto.

**Qué revisa la persona** en `docs/astra/launch.md`:

- [ ] La dirección de testnet registrada corresponde al mismo commit que se va a lanzar (o el devlog explica la diferencia).
- [ ] Todos los hallazgos críticos y altos de `audit.md` están cerrados y re-probados en testnet; los aceptados tienen justificación.
- [ ] El costo estimado (A10) está escrito: deploy, inicialización, reservas o rent, y las primeras operaciones. Y hay fondos para eso en el alias de mainnet.
- [ ] El alias de mainnet es distinto del de testnet (A2) y la clave nunca pasó por un chat, un archivo del repo ni un log.
- [ ] Plan de pausa / upgrade / rollback: quién puede ejecutarlo, en cuánto tiempo, y qué pasa con los fondos de los usuarios mientras tanto.
- [ ] Verificación pública prevista (A6): qué explorer, con qué comando, quién lo confirma.
- [ ] Prueba de humo prevista con el monto mínimo posible.
- [ ] Quién firma y cuándo.

**Cómo se registra**: en la cabecera de `launch.md`

```
firmado_por: Nombre Apellido
fecha_firma: 2026-09-04
costo_estimado: 12 XLM (deploy 3 + reservas 4 + humo 5)
alias_mainnet: prod-deployer
plan_rollback: pausa por admin en < 1 h; upgrade vía SEP-0049; sin migración de fondos
```

`astra check` exige `firmado_por` sin placeholders (`<...>`), `fecha_firma` con fecha ISO y `costo_estimado` no vacío.

**Si se rechaza**: vuelve a la fase que corresponda (Construcción si hay que cambiar código, Auditoría si faltan hallazgos por cerrar) y se anota en `launch.md`, sección "Intentos", la fecha y el motivo. Un Gate 2 rechazado no es un fracaso: es el protocolo funcionando.

## Composición con otro protocolo

Si el sprint también corre un protocolo general de software para la parte que no toca la cadena, sus gates humanos se alinean **el mismo día** con los de ASTRA: la persona revisa los dos planes juntos (Gate 1 con el plan de arquitectura; Gate 2 con el gate de release). ASTRA no necesita saber cuál es ese protocolo; solo que la persona no tenga que decidir dos veces lo mismo.
