# Glosario de ASTRA

Vocabulario del protocolo. Los términos se usan como se definen aquí en todos los documentos, plantillas y skills.

| Término | Definición |
|---|---|
| **ASTRA** | Protocolo de desarrollo Web3 agéntico: siete fases, dos gates humanos, diez axiomas. Del latín *astra*, los astros. |
| **Órbita** | Fase 1. Perfil de cadena, veredicto de capacidad, estándares candidatos y modelo de amenazas inicial. Artefacto: `docs/astra/orbit.md`. |
| **Carta** | Fase 2. El diseño completo del artefacto (interfaz, storage, roles, invariantes, economía, estándares, tests, upgrade). Artefacto: `docs/astra/chart.md`. La aprueba la persona en el Gate 1 con la línea `aprobada: YYYY-MM-DD`. |
| **Construcción** | Fase 3. Implementación con el toolchain nativo y tests locales verdes. |
| **Ensayo** | Fase 4. Deploy a testnet, integración contra la red real, conformidad con el estándar, costos medidos. |
| **Auditoría** | Fase 5. Revisión de seguridad por un rol distinto del autor (A9). Artefacto: `docs/astra/audit.md` con `veredicto: apto` o `no-apto`. |
| **Lanzamiento** | Fase 6. Deploy a mainnet firmado por la persona, verificación pública y registro. |
| **Bitácora** | Fase 7. Devlog del sprint y coherencia final de los artefactos. |
| **Gate 1 / Gate de la Carta** | Aprobación humana del diseño. Sin ella no hay Construcción. |
| **Gate 2 / Gate de mainnet** | Aprobación humana del lanzamiento. `astra check --gate mainnet` lo verifica mecánicamente; la persona firma `docs/astra/launch.md`. |
| **Axioma** | Regla que no se negocia dentro de un sprint (A1–A10). Cambia solo con bump MAJOR. |
| **Familia de cadena** | Grupo de cadenas que comparten formato de direcciones, forma de firma, sonda y verificación: `stellar`, `evm`, `utxo`. |
| **Perfil de cadena** | Entrada del registro `data/chains.json`: id, familia, red, chainId, CAIP-2, RPC, explorers, faucets, docs, notas, `verifiedAt`. |
| **Registro de cadenas** | `data/chains.json` de `astra-cli`, verificado en vivo con fecha. |
| **Registro de despliegues** | `.astra/deployments.json`: toda dirección desplegada con cadena, red, tx, commit, fecha y verificación (A4). |
| **Alias (de clave)** | Nombre bajo el cual un keystore nativo guarda una clave (`stellar keys`, `cast wallet`). El agente conoce el alias; nunca la clave (A2). |
| **Veredicto de capacidad** | Salida de `astra doctor`: SAFE (toolchain completo), CAUTION (se puede avanzar con límites), AVOID (instalar primero). |
| **Sonda** | `astra chain probe`: comprobación en vivo de que un RPC apunta a la red esperada (chainId o passphrase) y avanza. |
| **StrKey** | Codificación de claves y direcciones de Stellar (SEP-0023): base32 + byte de versión + CRC16. Prefijos G (cuenta), C (contrato), M (multiplexada), S (secreta). |
| **EIP-55** | Checksum de direcciones EVM por mayúsculas/minúsculas usando keccak-256. |
| **CAIP-2** | Identificador de cadena agnóstico: `eip155:8453`, `stellar:pubnet`. |
| **SAC** | Stellar Asset Contract: el contrato Soroban que representa un activo clásico de Stellar e implementa SEP-0041. |
| **TTL (storage)** | Tiempo de vida de una entrada de storage en Soroban; al expirar se archiva y hay que restaurarla. Se extiende con `extend_ttl` y cuesta rent. |
| **Reserva base** | 0.5 XLM por entrada de ledger que una cuenta Stellar debe mantener. |
| **NEVM** | La capa EVM de Syscoin (chainId 57; testnet Tanenbaum 5700). |
| **Rollux** | L2 optimista sobre OP Stack que asienta en Syscoin NEVM (chainId 570). |
| **PoDA** | Proof of Data Availability: capa de disponibilidad de datos de Syscoin que usan los rollups. |
| **OP Stack** | Conjunto de componentes de Optimism con el que se construyen L2 como Base y Rollux. |
| **Merge mining (AuxPoW)** | Minado conjunto: Syscoin L1 hereda hashrate de los mineros de Bitcoin. |
| **Facilitador (x402)** | Servicio que verifica y liquida en cadena un pago x402 en nombre del servidor; en Stellar además paga las fees del cliente. |
| **MPP** | Machine Payments Protocol: pagos máquina a máquina por cargo o por canal. |
| **Tier** | Nivel de modelo de IA sin nombrar vendor: `primary` (el más capaz) o `secondary` (económico). |
| **Runtime** | Herramienta de agente de IA que ejecuta el protocolo: Claude Code, Codex, Kimi Code, Cursor, Gemini CLI, Antigravity, OpenCode. |
| **Skill** | Instrucciones en formato Agent Skills (`<nombre>/SKILL.md`) que un runtime carga bajo demanda. Las canónicas viven en `skills/`; las copias son generadas. |
| **MCP** | Model Context Protocol: la forma en que `astra mcp` expone sus capacidades como tools a cualquier runtime. |
| **Regla de la caja** | Todo despacho a un sub-agente lleva objetivo, entradas, criterio de done y límite. |
| **Verificación pública** | Código fuente verificado en el explorer, o hash de WASM reproducible publicado, antes de anunciar una dirección (A6). |
| **Prueba de humo** | Primera operación real en mainnet con el monto mínimo, ejecutada por la persona. |
