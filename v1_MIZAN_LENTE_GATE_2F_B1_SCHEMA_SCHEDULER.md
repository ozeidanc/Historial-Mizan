MIZAN · LA LENTE
GATE 2F-B1 · INTEGRACIÓN OPERATIVA INACTIVA
SCHEMA v30 · LINEAGE PROSPECTIVO · DAILY-CLOSE · STARTUP CATCH-UP
SIN ENDPOINTS DE ESCRITURA · SIN UI · SIN ACTIVACIÓN

NATURALEZA

Esta ejecución autoriza:

1. finalizar el schema v30 para soportar posiciones prospectivas sin holdings
   sentinel;
2. hacer nullable paper_global_position.source_holding_id exclusivamente para
   posiciones prospectivas;
3. añadir lineage obligatorio desde una posición prospectiva hacia su
   allocation request y ejecución;
4. cablear la migración v29 → v30 en el runner canónico;
5. probar exhaustivamente la migración y el table rebuild en LAB;
6. integrar processPendingPaperExecutions con daily-close;
7. integrar la recuperación con startup catch-up;
8. implementar gates que impidan ejecutar mientras la política esté inactiva;
9. desplegar schema y código en Producción con la política todavía inactiva;
10. demostrar que la migración no crea reservas, posiciones, operaciones ni
    snapshots;
11. crear un único commit local.

No autoriza:

- activar MANUAL_NOTIONAL_HARD_CAP;
- fijar effective_from;
- crear endpoints públicos de escritura;
- crear allocation requests productivas;
- crear position action requests productivas;
- modificar la UI;
- conectar acciones desde tesis;
- conectar alertas de catalizador con botones;
- crear carteras nuevas;
- crear aportaciones externas;
- ejecutar compras productivas;
- ejecutar ventas productivas;
- ejecutar reducciones productivas;
- crear datos de prueba productivos;
- utilizar holdings sentinel;
- modificar tesis;
- modificar catalizadores;
- modificar Campo de Caza;
- modificar Book, cash o NAV reales;
- modificar el Track histórico;
- hacer push;
- crear tag;
- realizar operaciones Git remotas.

El código de ejecución debe quedar desplegado pero deshabilitado mediante el
estado y effective_from de la política.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = 88892cd
schema de Producción esperado = v29
schema objetivo = v30
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Gate 2F-A1:

- lifecycle append-only;
- cash reservations;
- quantity reservations;
- capacity race protection;
- quantity race protection;
- catalyst reconciliation 7/7;
- 101 PASS / 0 FAIL.

Gate 2F-A2:

- ejecución EOD de compras;
- ejecución EOD de asignaciones parciales;
- ejecución EOD de reducciones;
- ejecución EOD de ventas completas;
- PENDING_MARKET_CLOSE;
- PENDING_PRICE fail-closed;
- neutralidad NAV;
- neutralidad Unit Value;
- consumo atómico de reservas;
- idempotencia;
- concurrencia;
- procesamiento cronológico;
- 132 PASS / 0 FAIL.

Producción:

PAPER_PORTFOLIOS = 5
PAPER_GLOBAL_POSITIONS = 82
PAPER_MEMBERSHIP_EPISODES = 121
PAPER_TRACK_SNAPSHOTS = 54

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

Debe permanecer así.

==================================================
1. DEFECTO DEL SENTINEL
==================================================

Gate 2F-A2 identificó:

paper_global_position.source_holding_id

actualmente es:

NOT NULL
UNIQUE
FOREIGN KEY hacia holdings legacy.

Una posición prospectiva no tiene un holding legacy de origen.

Está prohibido resolver esto mediante:

- fila holdings __PROSPECTIVE__;
- ticker ficticio;
- holding placeholder;
- reutilización de otro source_holding_id;
- creación de holdings legacy por una operación papel;
- uso de un ID negativo o inválido.

La solución debe representar directamente la semántica correcta:

POSICIÓN HISTÓRICA

- source_holding_id NOT NULL;
- prospective_allocation_request_id NULL;
- prospective_execution_id NULL, salvo referencia informativa adicional.

POSICIÓN PROSPECTIVA

- source_holding_id NULL;
- prospective_allocation_request_id NOT NULL;
- prospective_execution_id NOT NULL después de ejecutarse;
- source_type = PROSPECTIVE_ALLOCATION.

Debe existir exactamente una fuente de lineage primaria válida.

==================================================
2. INVARIANTE DE LINEAGE
==================================================

Aplicar una restricción equivalente a:

HISTORICAL:

source_type = 'LEGACY_HOLDING'
AND source_holding_id IS NOT NULL
AND prospective_allocation_request_id IS NULL

PROSPECTIVE:

source_type = 'PROSPECTIVE_ALLOCATION'
AND source_holding_id IS NULL
AND prospective_allocation_request_id IS NOT NULL

No permitir:

- las dos fuentes nulas;
- las dos fuentes pobladas;
- source_type incompatible;
- posición prospectiva sin request;
- posición histórica sin holding.

Si SQLite no puede expresar toda la restricción con foreign keys:

- usar CHECK;
- índices únicos parciales;
- validación transaccional;
- pruebas de invariantes.

La ejecución prospectiva debe vincular además:

allocation request
→ reservation
→ prospective execution
→ global position
→ membership
→ composition
→ transaction
→ snapshot

==================================================
3. MIGRACIÓN DE PAPER_GLOBAL_POSITION
==================================================

SQLite puede requerir reconstruir la tabla.

Aplicar en LAB:

1. BEGIN IMMEDIATE;
2. comprobar user_version = 29;
3. desactivar foreign keys únicamente dentro del procedimiento canónico y
   seguro si el runner lo exige;
4. crear paper_global_position_v30;
5. copiar las 82 filas históricas;
6. asignar source_type = LEGACY_HOLDING;
7. conservar IDs;
8. conservar todos los campos;
9. conservar timestamps;
10. conservar content hashes o recalcular únicamente si el contrato lo exige;
11. renombrar tablas;
12. recrear índices;
13. recrear foreign keys;
14. recrear triggers;
15. crear tablas v30 de Gate 2F-A1/A2;
16. establecer user_version = 30;
17. activar y comprobar foreign keys;
18. ejecutar foreign_key_check;
19. ejecutar integrity_check;
20. COMMIT.

Si falla:

- ROLLBACK;
- schema permanece v29;
- 82 posiciones siguen intactas;
- no quedan tablas temporales.

No eliminar la tabla anterior antes de comprobar que la copia reconcilia.

==================================================
4. PRESERVACIÓN DE LAS 82 POSICIONES
==================================================

Después de migrar:

PAPER_GLOBAL_POSITIONS = 82

Para las 82:

source_type =
LEGACY_HOLDING

source_holding_id =
valor original

prospective_allocation_request_id =
NULL

Conservar exactamente:

- global_position_id;
- ticker;
- thesis_id;
- incorporated_at;
- status;
- entry data;
- lineage;
- relaciones con 121 memberships;
- relaciones con composiciones;
- relaciones con transacciones;
- relaciones con snapshots;
- hashes económicos.

Comparar antes/después:

- contenido normalizado;
- foreign keys;
- memberships;
- composiciones;
- Track.

No aceptar pérdida ni reasignación de IDs.

==================================================
5. POSICIÓN PROSPECTIVA SIN SENTINEL
==================================================

Modificar el motor LAB de Gate 2F-A2.

Al ejecutar NEW_POSITION:

- no crear holding legacy;
- no insertar fila sentinel;
- no modificar holdings;
- crear paper_global_position con:
  - source_type = PROSPECTIVE_ALLOCATION;
  - source_holding_id = NULL;
  - prospective_allocation_request_id = request;
  - ticker;
  - thesis_id;
  - incorporated_at;
  - status = OPEN;
  - lineage hash.

Después de crear la ejecución:

prospective_execution_id =
execution canónica correspondiente.

Debe cumplirse:

PROSPECTIVE_SENTINEL_HOLDINGS_CREATED = 0

LEGACY_HOLDINGS_CREATED_BY_PAPER_EXECUTION = 0

==================================================
6. SCHEMA v30 CANÓNICO
==================================================

Consolidar en una única migración v29 → v30:

- request lifecycle;
- position action lifecycle;
- cash reservations;
- quantity reservations;
- policy activation audit;
- prospective execution;
- lineage prospectivo de global positions;
- índices;
- constraints;
- triggers append-only, si existen.

No mantener dos definiciones incompatibles de v30.

No crear v31 porque v30 todavía no se ha aplicado en Producción.

Actualizar:

- migration registry;
- db-version;
- schema verification;
- repository sweep;
- rollback/forward-only documentation;
- test helpers.

La migración debe ser determinista e idempotente según el contrato del runner.

==================================================
7. DEUDA DE TEST PREEXISTENTE
==================================================

Gate 2F-A2 informó que:

verify-paper-economic-reconstruction-version.mjs

falla al acceder:

nav_basis

sobre undefined.

Antes del despliegue de v30:

- diagnosticar si el fixture está obsoleto;
- corregir únicamente el test o fixture si la implementación productiva es
  correcta;
- no ocultar una regresión;
- no desactivar la suite;
- no convertir el error en skip silencioso.

Entregar:

ECONOMIC_RECONSTRUCTION_VERSION_TEST =
PASS /
PREEXISTING_BLOCKER

No desplegar v30 si esta prueba cubre una invariante afectada por la migración
y sigue fallando.

==================================================
8. GATE DE POLÍTICA PARA EL SCHEDULER
==================================================

Crear una función canónica:

isProspectivePaperExecutionEnabled

Debe devolver true únicamente cuando:

- policy = MANUAL_NOTIONAL_HARD_CAP;
- status = ACTIVE;
- effective_from no es NULL;
- effective_session >= effective_from;
- configuración de cartera vigente;
- schema soportado;
- ejecución no suspendida.

Con el estado actual:

APPROVED_PENDING_ACTIVATION
effective_from = NULL

debe devolver:

false

daily-close y catch-up pueden invocar el adaptador, pero este debe terminar:

POLICY_NOT_ACTIVE

sin escribir.

No ejecutar por la mera existencia de:

- reserva;
- request;
- tabla v30;
- código desplegado;
- tesis aprobada;
- alerta de catalizador.

==================================================
9. DAILY-CLOSE
==================================================

Integrar en el orquestador existente mediante un adaptador separado:

persistProspectivePaperExecutions

Orden recomendado:

1. resolver sesión cerrada;
2. ejecutar Track real existente;
3. actualizar precios/Track Papel existente;
4. comprobar policy gate;
5. si inactiva → POLICY_NOT_ACTIVE y detener rama prospectiva;
6. si activa → localizar ejecuciones pendientes;
7. extender manifiesto;
8. procesar reservas por cartera y sesión;
9. crear composición/snapshot;
10. registrar resultado.

No crear un segundo scheduler.

No mezclar el camino papel con:

- Book real;
- cash real;
- NAV real;
- operaciones reales.

El fallo prospectivo:

- no bloquea Track real;
- no bloquea Track Papel diario ya existente;
- no bloquea otras carteras;
- queda auditado;
- puede recuperarse mediante catch-up.

==================================================
10. STARTUP CATCH-UP
==================================================

Integrar:

catchUpProspectivePaperExecutions

Al arrancar:

1. comprobar policy gate;
2. si inactiva → POLICY_NOT_ACTIVE;
3. si activa:
   - buscar ejecuciones pendientes;
   - ordenar por cartera y sesión;
   - procesar cronológicamente;
   - detener una cartera ante un hueco;
   - continuar otras carteras;
   - no duplicar;
   - no usar precio live.

Debe recuperar:

- cierre ocurrido con servidor apagado;
- precio publicado tarde;
- reinicio antes del COMMIT;
- reinicio después del COMMIT;
- ejecución en PENDING_PRICE;
- snapshot pendiente.

No puede:

- crear requests;
- confirmar requests;
- reservar cash;
- reservar cantidad;
- activar la política.

==================================================
11. MANIFIESTO Y PRECIOS
==================================================

Las ejecuciones prospectivas deben usar la extensión versionada de Gate 2E-A.

Para cada sesión/ticker:

- OFFICIAL_CLOSE_NOMINAL;
- proveedor canónico;
- evidencia durable;
- manifest version;
- content hash.

Si falta un precio:

- PENDING_PRICE;
- reserva sigue ACTIVE;
- no se crea transacción;
- no se crea composición;
- no se modifica snapshot.

El GET y la UI nunca obtienen precios.

==================================================
12. DESPLIEGUE INACTIVO
==================================================

El despliegue autorizado en este gate incluye:

- migración v30;
- código de servicios;
- integración daily-close;
- integración catch-up.

Pero debe mantener:

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

Resultado esperado tras desplegar:

- schema v30;
- cero requests prospectivas productivas;
- cero position actions productivas;
- cero reservas productivas;
- cero executions productivas;
- cero posiciones nuevas;
- cero memberships nuevos;
- cero transacciones prospectivas;
- cero snapshots creados por la rama prospectiva;
- scheduler devuelve POLICY_NOT_ACTIVE.

No crear datos ficticios para probar Producción.

==================================================
13. PREVIEW DE PRODUCCIÓN
==================================================

Antes de migrar Producción:

- backup SQLite;
- SHA-256;
- abrir backup;
- integrity_check;
- foreign_key_check;
- contadores;
- hashes;
- migrar una copia exacta;
- ejecutar todas las suites;
- iniciar servidor contra la copia;
- ejecutar daily-close;
- ejecutar startup catch-up;
- verificar POLICY_NOT_ACTIVE;
- comparar datos económicos.

No migrar Producción si:

- falla table rebuild;
- cambia algún ID;
- cambia un hash económico sin justificación;
- aparece un FK huérfano;
- el scheduler escribe con la política inactiva;
- las suites no están verdes.

==================================================
14. TRANSACCIÓN DE MIGRACIÓN PRODUCTIVA
==================================================

Antes:

- detener escrituras;
- una sola instancia;
- backup validado;
- baseline completo.

Aplicar v29 → v30.

Después:

- schema v30;
- integrity_check;
- foreign_key_check;
- 82 global positions;
- 121 memberships;
- 54 snapshots o los que correspondan únicamente por frescura normal;
- cero datos prospectivos nuevos;
- policy inactiva.

Si falla:

- rollback;
- restaurar servicio sobre v29;
- no activar política;
- no continuar.

==================================================
15. PRUEBAS
==================================================

Crear:

verify-paper-v30-global-position-lineage.mjs
verify-paper-v30-position-migration.mjs
verify-paper-v30-no-sentinel-holding.mjs
verify-paper-prospective-policy-gate.mjs
verify-paper-prospective-daily-close.mjs
verify-paper-prospective-startup-catchup.mjs
verify-paper-prospective-pending-price-integration.mjs
verify-paper-prospective-scheduler-isolation.mjs
verify-paper-v30-production-inactive.mjs

Casos obligatorios:

A. 82 posiciones migradas.
B. IDs preservados.
C. Histórico requiere source_holding.
D. Prospectivo requiere allocation request.
E. Ambas fuentes nulas falla.
F. Ambas fuentes pobladas falla.
G. Posición prospectiva sin sentinel.
H. Holdings legacy no aumentan.
I. Gate inactivo devuelve false.
J. Gate activo antes de effective_from devuelve false.
K. Gate activo desde effective_from devuelve true.
L. Daily-close inactivo no escribe.
M. Catch-up inactivo no escribe.
N. PENDING_PRICE conserva reserva.
O. Daily-close y catch-up no duplican.
P. Fallo prospectivo no afecta Track real.
Q. Fallo prospectivo no afecta Track Papel ordinario.
R. Migración idempotente.
S. Foreign keys limpias.
T. Integrity check.
U. Suite de economic reconstruction verde.
V. Producción inactiva tras despliegue.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- 101 pruebas Gate 2F-A1;
- 132 pruebas Gate 2F-A2;
- suites v27/v28/v29;
- migración v30;
- Gate 2D-2;
- Gate 2E-A;
- financial core;
- daily-close;
- startup catch-up;
- repository integrity.

==================================================
16. AISLAMIENTO
==================================================

Comparar antes/después:

- holdings legacy;
- global positions;
- memberships;
- composiciones históricas;
- transacciones históricas;
- snapshots;
- Track Papel;
- gráficas;
- tesis;
- catalizadores;
- alertas;
- Campo de Caza;
- Book real;
- cash real;
- NAV real;
- valuations reales.

Resultados:

LEGACY_HOLDINGS_UNCHANGED = PASS
HISTORICAL_GLOBAL_POSITIONS_UNCHANGED = PASS
HISTORICAL_MEMBERSHIPS_UNCHANGED = PASS
HISTORICAL_COMPOSITIONS_UNCHANGED = PASS
HISTORICAL_TRANSACTIONS_UNCHANGED = PASS
HISTORICAL_SNAPSHOTS_UNCHANGED = PASS
PAPER_TRACK_ECONOMICALLY_UNCHANGED = PASS
REAL_BOOK_UNCHANGED = PASS
REAL_CASH_UNCHANGED = PASS
REAL_NAV_UNCHANGED = PASS
THESES_UNCHANGED = PASS
CATALYSTS_UNCHANGED = PASS
CATALYST_ALERTS_UNCHANGED = PASS

==================================================
17. CÓDIGO Y COMMIT
==================================================

Añadir:

- migración v30 canónica;
- lineage prospectivo;
- eliminación del sentinel;
- policy gate;
- adaptador daily-close;
- adaptador startup catch-up;
- integración de manifiesto;
- pruebas;
- documentación de rollback.

No añadir:

- endpoints públicos de escritura;
- UI;
- carteras nuevas;
- aportaciones;
- activación de política;
- requests productivas;
- datos ficticios;
- scratchpad;
- backups;
- bases;
- secretos.

Ejecutar secret scan.

Crear un único commit local:

feat(lens): integrate inactive prospective paper execution

No crear tag.
No hacer push.

==================================================
18. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = 88892cd
CANONICAL_HEAD_AFTER = <hash>

SCHEMA_BEFORE = v29
SCHEMA_AFTER = v30

V30_GLOBAL_POSITION_REBUILT = PASS/FAIL
HISTORICAL_POSITION_IDS_PRESERVED = <número>/82
HISTORICAL_POSITION_LINEAGE_VALID = <número>/82

PROSPECTIVE_SOURCE_HOLDING_NULLABLE = PASS/FAIL
PROSPECTIVE_REQUEST_LINEAGE_REQUIRED = PASS/FAIL
PROSPECTIVE_SENTINEL_HOLDINGS_CREATED = 0
LEGACY_HOLDINGS_CREATED_BY_PAPER_EXECUTION = 0

V30_TABLES = <lista>
V30_MIGRATION_TRANSACTIONAL = PASS/FAIL
V30_MIGRATION_IDEMPOTENT = PASS/FAIL
V30_FOREIGN_KEYS = PASS/FAIL
V30_INTEGRITY = PASS/FAIL

ECONOMIC_RECONSTRUCTION_VERSION_TEST = PASS/FAIL

PROSPECTIVE_POLICY_GATE = PASS/FAIL
DAILY_CLOSE_PROSPECTIVE_ADAPTER = PASS/FAIL
STARTUP_CATCHUP_PROSPECTIVE = PASS/FAIL
PENDING_PRICE_INTEGRATION = PASS/FAIL
MANIFEST_EXTENSION_INTEGRATION = PASS/FAIL

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

POLICY_NOT_ACTIVE_DAILY_CLOSE = PASS/FAIL
POLICY_NOT_ACTIVE_CATCHUP = PASS/FAIL

PRODUCTION_PROSPECTIVE_REQUESTS = 0
PRODUCTION_POSITION_ACTIONS = 0
PRODUCTION_CASH_RESERVATIONS = 0
PRODUCTION_QUANTITY_RESERVATIONS = 0
PRODUCTION_EXECUTIONS = 0
PRODUCTION_PROSPECTIVE_POSITIONS = 0
PRODUCTION_PROSPECTIVE_TRANSACTIONS = 0
PRODUCTION_POLICY_ACTIVATION_ROWS = 0

POLICY_INACTIVE_WRITES = 0

LEGACY_HOLDINGS_UNCHANGED = PASS/FAIL
HISTORICAL_GLOBAL_POSITIONS_UNCHANGED = PASS/FAIL
HISTORICAL_MEMBERSHIPS_UNCHANGED = PASS/FAIL
HISTORICAL_COMPOSITIONS_UNCHANGED = PASS/FAIL
HISTORICAL_TRANSACTIONS_UNCHANGED = PASS/FAIL
HISTORICAL_SNAPSHOTS_UNCHANGED = PASS/FAIL
PAPER_TRACK_ECONOMICALLY_UNCHANGED = PASS/FAIL

REAL_BOOK_UNCHANGED = PASS/FAIL
REAL_CASH_UNCHANGED = PASS/FAIL
REAL_NAV_UNCHANGED = PASS/FAIL
THESES_UNCHANGED = PASS/FAIL
CATALYSTS_UNCHANGED = PASS/FAIL
CATALYST_ALERTS_UNCHANGED = PASS/FAIL

SECRET_SCAN = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

LOCAL_COMMIT = <hash>

EXACT_BLOCKERS_BEFORE_GATE_2F_B2:
1. endpoints de preview/create/confirm/cancel;
2. creación dinámica de carteras;
3. UI desde tesis y alertas.

Detenerse después de Gate 2F-B1.