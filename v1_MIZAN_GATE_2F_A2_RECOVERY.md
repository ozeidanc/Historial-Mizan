MIZAN · LA LENTE
GATE 2F-A2 · RECUPERACIÓN Y CIERRE CONTROLADO

Code se detuvo durante Gate 2F-A2.

No reiniciar desde cero.
No descartar cambios útiles.
No resetear el repositorio.
No duplicar tablas, servicios o pruebas.
No aplicar v30 en Producción.
No activar MANUAL_NOTIONAL_HARD_CAP.
No ejecutar operaciones productivas.
No hacer push ni tag.

==================================================
1. RECUPERAR EL ESTADO EXACTO
==================================================

Baseline anterior:

CANONICAL_HEAD_ORIGINAL = b651c7e
PRODUCTION_SCHEMA = v29
LAB_SCHEMA_TARGET = v30

Auditar y reportar:

- HEAD actual;
- git status;
- archivos tracked modificados;
- archivos untracked;
- commits creados;
- cambios realizados sobre Gate 2F-A1;
- ampliaciones del schema v30;
- tablas de ejecución creadas;
- servicios implementados;
- pruebas creadas;
- pruebas ejecutadas;
- último paso completado;
- primer paso pendiente;
- causa exacta de la detención.

No asumir que los archivos untracked son desechables.

Verificar Producción:

- schema v29;
- integrity_check;
- foreign_key_check;
- hard cap APPROVED_PENDING_ACTIVATION;
- effective_from NULL;
- cero tablas v30 productivas;
- cero reservas productivas;
- cero ejecuciones productivas;
- cero compras o ventas prospectivas;
- snapshots productivos intactos;
- Track Papel intacto;
- tesis, catalizadores y alertas intactos;
- Book, cash y NAV reales intactos.

Si Producción fue modificada accidentalmente:

- detenerse;
- no intentar corregir silenciosamente;
- entregar diagnóstico y diferencias exactas.

==================================================
2. CLASIFICAR LA RECUPERACIÓN
==================================================

RECOVERY_STATUS debe ser uno de:

- NO_CHANGES;
- PARTIAL_UNCOMMITTED;
- PARTIAL_COMMITTED;
- IMPLEMENTATION_COMPLETE_TESTS_PENDING;
- COMPLETE.

Si existe un commit parcial:

- conservarlo;
- no resetearlo;
- informar el hash;
- continuar desde ese HEAD.

Si existen cambios útiles sin commit:

- inspeccionarlos;
- continuar sobre ellos;
- no crear implementaciones paralelas.

Si el schema v30 fue ampliado:

- conservar una sola definición canónica;
- no crear v31;
- recordar que v30 sigue siendo LAB-only;
- mantener compatibilidad con las 101 pruebas de Gate 2F-A1.

==================================================
3. DIVIDIR AUTOMÁTICAMENTE SI ES NECESARIO
==================================================

Si 2F-A2 completo sigue siendo demasiado grande, dividirlo sin pedir permiso:

SUBGATE 2F-A2.1 · EJECUCIÓN DE COMPRAS

- soporte de ejecución;
- effective session;
- PENDING_MARKET_CLOSE;
- PENDING_PRICE;
- compra completa;
- compra parcial;
- posición global;
- membership;
- composición EOD;
- transacción PURCHASE;
- snapshot;
- neutralidad NAV/Unit Value;
- consumo atómico de cash reservation;
- idempotencia;
- concurrencia;
- aislamiento.

SUBGATE 2F-A2.2 · EJECUCIÓN DE SALIDAS

- reducción parcial;
- venta total;
- consumo de quantity reservation;
- Paper Cash;
- membership abierto/cerrado;
- alertas de catalizador;
- neutralidad NAV/Unit Value;
- idempotencia;
- concurrencia;
- venta no autoejecuta compra pendiente;
- aislamiento.

Completar primero 2F-A2.1.

Si no puede completarse todo 2F-A2 en esta sesión:

- dejar 2F-A2.1 completamente funcional y verde;
- crear un commit local coherente;
- entregar checkpoint exacto;
- no dejar código de ventas parcialmente funcional;
- no dejar una migración inconsistente.

==================================================
4. PRIORIDAD DE RECUPERACIÓN
==================================================

Orden:

1. comprobar que Gate 2F-A1 sigue con 101 PASS / 0 FAIL;
2. validar la definición única de schema v30;
3. terminar el modelo de ejecución;
4. terminar resolución de effective session;
5. terminar PENDING_MARKET_CLOSE;
6. terminar PENDING_PRICE fail-closed;
7. terminar compra completa;
8. terminar compra parcial;
9. demostrar neutralidad NAV/Unit Value;
10. demostrar consumo atómico de reserva;
11. demostrar snapshot único;
12. demostrar idempotencia/concurrencia;
13. después abordar reducción y venta;
14. ejecutar todas las regresiones.

No empezar HTTP, UI, scheduler productivo ni activación.

==================================================
5. REGLAS ECONÓMICAS QUE NO PUEDEN CAMBIAR
==================================================

COMPRA

executed_notional =
cash reservation amount.

executed_quantity =
executed_notional / official close.

La compra:

- reduce Paper Cash;
- aumenta Paper Market Value;
- no modifica Paper NAV;
- no modifica Paper Units;
- no modifica Paper Unit Value;
- no crea retorno;
- no rebalancea posiciones existentes.

REDUCCIÓN/VENTA

executed_notional =
reserved quantity × official close.

La venta:

- reduce Paper Market Value;
- aumenta Paper Cash;
- no modifica Paper NAV;
- no modifica Paper Units;
- no modifica Paper Unit Value;
- no crea retorno adicional;
- no cambia SPY.

Una venta reservada pero no ejecutada:

- no crea Paper Cash;
- no aumenta capacidad;
- no activa una compra pendiente.

Una venta ejecutada:

- crea Paper Cash;
- pero no confirma ni ejecuta automáticamente otra solicitud.

==================================================
6. GATES DE SEGURIDAD
==================================================

Mantener:

PRODUCTION_READ_ONLY = true
LAB_ONLY = true
POLICY_ACTIVATION_ALLOWED = false
PUBLIC_HTTP_ALLOWED = false
PRODUCTION_SCHEDULER_INTEGRATION_ALLOWED = false
PRODUCTION_UI_CHANGES_ALLOWED = false

No modificar:

- db.js productivo;
- db-version productivo;
- schema de Producción;
- daily-close productivo;
- startup catch-up productivo;
- endpoints;
- UI;
- snapshots históricos;
- composiciones históricas;
- transacciones históricas;
- tesis;
- catalizadores;
- alertas;
- patrimonio real.

==================================================
7. PRECIO Y SESIÓN
==================================================

Usar únicamente:

PRICE_BASIS =
OFFICIAL_CLOSE_NOMINAL

La effective session debe respetar:

- America/New_York;
- sesión abierta;
- after-close;
- early close;
- festivo;
- fin de semana.

Si falta precio exacto de effective_session:

- PENDING_PRICE;
- reserva sigue ACTIVE;
- cero operación;
- cero composición;
- cero snapshot.

No utilizar:

- live;
- apertura;
- previous close;
- adjusted total-return;
- precio de otra sesión.

==================================================
8. TRANSACCIONALIDAD
==================================================

La ejecución completa debe ocurrir en una sola transacción:

- validar reserva;
- validar request/action;
- validar precio;
- crear o actualizar posición/membership;
- crear transacción;
- crear composición EOD;
- crear snapshot;
- consumir reserva;
- actualizar lifecycle;
- COMMIT.

Si falla:

- ROLLBACK;
- reserva permanece ACTIVE;
- cero efecto económico parcial.

Una ejecución repetida:

- devuelve ALREADY_EXECUTED;
- no duplica ninguna fila.

==================================================
9. ALERTAS DE CATALIZADOR
==================================================

Mantener reconciliadas:

- ONB;
- ATRC;
- ISRG;
- HCSG;
- WSBC;
- WTFC;
- PEGA.

En 2F-A2.2 probar en LAB:

- HOLD;
- PARTIAL_EXIT;
- FULL_EXIT;
- cancelación;
- PENDING_PRICE;
- doble ejecución.

No ejecutar estas operaciones en Producción.

No modificar el estado del catalizador por ejecutar una venta papel.

==================================================
10. PRUEBAS MÍNIMAS PARA 2F-A2.1
==================================================

Antes de considerar cerrado el subgate de compras:

- Gate 2F-A1 = 101 PASS / 0 FAIL;
- migración v30 verde;
- effective session verde;
- PENDING_MARKET_CLOSE verde;
- PENDING_PRICE verde;
- compra completa verde;
- compra parcial verde;
- global position creada una vez;
- membership creado una vez;
- composición creada una vez;
- transacción creada una vez;
- snapshot creado una vez;
- NAV continuity verde;
- Unit continuity verde;
- SPY sin cambios por compra;
- posiciones existentes sin rebalanceo;
- reserva consumida atómicamente;
- ejecución idempotente;
- dos workers no duplican;
- Producción intacta.

==================================================
11. PRUEBAS MÍNIMAS PARA 2F-A2.2
==================================================

Antes de cerrar salidas:

- reducción parcial mantiene membership abierto;
- venta total cierra membership;
- global position histórica permanece;
- proceeds van a Paper Cash;
- SPY no cambia;
- NAV continuity;
- Unit continuity;
- quantity reservation consumida atómicamente;
- venta reservada no crea cash;
- venta ejecutada no autoejecuta compra;
- cancelación frente a ejecución;
- doble worker no duplica;
- alertas 7/7 reconciliadas;
- Producción intacta.

==================================================
12. COMMIT
==================================================

No crear commit vacío.

Si se completa solo 2F-A2.1:

feat(lens): add NAV-neutral paper purchase execution

Si se completa todo 2F-A2:

feat(lens): add NAV-neutral paper EOD execution engine

Si ya existe un commit parcial:

- conservarlo;
- informar su hash;
- no duplicar cambios;
- crear otro commit solo para una unidad coherente pendiente.

No crear tag.
No hacer push.

==================================================
13. ENTREGA
==================================================

RECOVERY_STATUS =
NO_CHANGES /
PARTIAL_UNCOMMITTED /
PARTIAL_COMMITTED /
IMPLEMENTATION_COMPLETE_TESTS_PENDING /
COMPLETE

CANONICAL_HEAD_ORIGINAL = b651c7e
HEAD_FOUND_AT_RECOVERY = <hash>
CANONICAL_HEAD_AFTER = <hash/UNCHANGED>

PRODUCTION_SCHEMA = v29
LAB_SCHEMA = <v29/v30>

PRODUCTION_READ_ONLY = PASS/FAIL
PRODUCTION_DATA_UNCHANGED = PASS/FAIL
NO_POLICY_ACTIVATED = PASS/FAIL
NO_ECONOMIC_PRODUCTION_EXECUTION = PASS/FAIL

FILES_FOUND_MODIFIED = <lista>
FILES_FOUND_UNTRACKED = <lista>
COMMITS_FOUND = <lista>

LAST_COMPLETED_STEP = <paso>
FIRST_PENDING_STEP = <paso>
STOP_REASON = <causa>

GATE_2F_A1_REGRESSION = PASS/FAIL
V30_EXECUTION_SUPPORT = NOT_STARTED/PARTIAL/PASS
EFFECTIVE_SESSION_RESOLUTION = NOT_STARTED/PARTIAL/PASS
PENDING_MARKET_CLOSE = NOT_STARTED/PARTIAL/PASS
PENDING_PRICE = NOT_STARTED/PARTIAL/PASS

NEW_POSITION_EXECUTION = NOT_STARTED/PARTIAL/PASS
PARTIAL_ALLOCATION_EXECUTION = NOT_STARTED/PARTIAL/PASS
PURCHASE_NAV_NEUTRAL = NOT_STARTED/PARTIAL/PASS
PURCHASE_UNIT_NEUTRAL = NOT_STARTED/PARTIAL/PASS
CASH_RESERVATION_CONSUMED_ATOMICALLY = NOT_STARTED/PARTIAL/PASS

PARTIAL_EXIT_EXECUTION = NOT_STARTED/PARTIAL/PASS
FULL_EXIT_EXECUTION = NOT_STARTED/PARTIAL/PASS
EXIT_NAV_NEUTRAL = NOT_STARTED/PARTIAL/PASS
EXIT_UNIT_NEUTRAL = NOT_STARTED/PARTIAL/PASS
QUANTITY_RESERVATION_CONSUMED_ATOMICALLY = NOT_STARTED/PARTIAL/PASS

EXECUTION_IDEMPOTENCY = NOT_STARTED/PARTIAL/PASS
EXECUTION_CONCURRENCY = NOT_STARTED/PARTIAL/PASS
CANCELLATION_EXECUTION_RACE = NOT_STARTED/PARTIAL/PASS

CATALYST_ALERTS_RECONCILED = <número>/7

TESTS_PASS = <número>
TESTS_FAIL = <número>
TESTS_PENDING = <lista>

LOCAL_COMMIT = <hash/NONE>

COMPLETED_SUBGATE =
NONE /
2F-A2.1 /
2F-A2.COMPLETE

EXACT_NEXT_STEP =
<una acción concreta>

Si el alcance completo no cabe, detenerse solamente después de dejar
2F-A2.1 completo, probado, coherente y commiteado.