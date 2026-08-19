MIZAN · LA LENTE
GATE 2F-A1 · FUNDACIÓN TRANSACCIONAL DEL HARD CAP
SCHEMA v30 · LIFECYCLE · CAPACIDAD · RESERVAS · CONCURRENCIA
LAB ÚNICAMENTE · POLÍTICA INACTIVA · SIN EJECUCIÓN ECONÓMICA

NATURALEZA

Esta ejecución autoriza exclusivamente:

1. diseñar la ampliación mínima del schema para el flujo prospectivo;
2. crear una migración v29 → v30;
3. probarla únicamente en copias LAB;
4. implementar el lifecycle de allocation requests;
5. implementar el cálculo canónico de capacidad disponible;
6. implementar reservas atómicas de Paper Cash;
7. implementar confirmación completa;
8. implementar confirmación parcial;
9. implementar cancelación y liberación de reservas;
10. implementar protección contra concurrencia y doble confirmación;
11. crear servicios internos y pruebas económicas;
12. crear un único commit local si todo queda verde.

No autoriza:

- aplicar v30 en Producción;
- activar MANUAL_NOTIONAL_HARD_CAP;
- fijar effective_from;
- crear compras o ventas;
- crear composiciones prospectivas;
- crear memberships prospectivos;
- crear posiciones globales prospectivas;
- crear snapshots;
- modificar daily-close;
- modificar startup catch-up;
- crear endpoints HTTP de escritura;
- modificar UI;
- crear carteras nuevas;
- crear aportaciones;
- crear retiradas;
- crear salidas de posiciones;
- crear reasignaciones prospectivas;
- modificar el Track Papel;
- modificar las gráficas;
- modificar datos históricos;
- modificar Book, cash o NAV reales;
- modificar tesis o catalizadores;
- desplegar;
- hacer push;
- crear tag;
- realizar operaciones Git remotas.

Toda prueba que requiera escrituras debe ejecutarse en una base LAB aislada.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = 0466595
schema de Producción esperado = v29
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Estado esperado:

PAPER_PORTFOLIOS = 5
PAPER_GLOBAL_POSITIONS = 82
PAPER_MEMBERSHIP_EPISODES = 121
PAPER_TRACK_SNAPSHOTS = 54

PAPER_TRACK_METHOD =
PRICE_RETURN_TWR

PROSPECTIVE_ALLOCATION_POLICY =
MANUAL_NOTIONAL_HARD_CAP

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

PROSPECTIVE_AUTO_REBALANCE =
DISABLED

PARTIAL_ALLOCATION_POLICY =
REQUIRE_EXPLICIT_CONFIRMATION

LEVERAGE_POLICY =
DISABLED

Las carteras existentes tienen Paper Cash aproximadamente igual a cero.

Por tanto, una solicitud prospectiva normal debe quedar:

PENDING_CAPITAL

salvo que exista Paper Cash disponible demostrado.

==================================================
1. OBJETIVO FUNCIONAL DE 2F-A1
==================================================

Implementar y demostrar este flujo interno en LAB:

DRAFT
→ AWAITING_CONFIRMATION
→ CONFIRMED
→ CAPACITY_CHECK
→ RESERVED_FOR_EXECUTION

o, si no hay capacidad:

DRAFT
→ AWAITING_CONFIRMATION
→ CONFIRMED
→ PENDING_CAPITAL

Asignación parcial:

PENDING_CAPITAL
→ PENDING_PARTIAL_CONFIRMATION
→ confirmación explícita
→ PARTIALLY_RESERVED_FOR_EXECUTION

Cancelación:

estado cancelable
→ CANCELLED
→ reserva liberada

Gate 2F-A1 termina en la reserva.

No ejecuta la compra.

No crea exposición.

No modifica el Track.

==================================================
2. PRECONDICIONES
==================================================

Antes de modificar código:

- confirmar HEAD 0466595;
- confirmar schema v29;
- confirmar env-info;
- confirmar una sola raíz Git;
- inventariar archivos untracked;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- confirmar health;
- confirmar política hard cap inactiva;
- capturar contadores e hashes productivos;
- crear una copia LAB validada de la base;
- ejecutar integrity_check y foreign_key_check sobre LAB.

Capturar baseline de:

- paper_allocation_policy;
- paper_portfolio_track_config;
- paper_allocation_request;
- paper_portfolio_flow;
- paper_portfolio_transaction;
- paper_composition_version;
- paper_composition_member;
- paper_track_snapshot;
- paper_economic_*;
- carteras_papel;
- holdings legacy;
- Track Papel;
- Book, cash y NAV reales.

No continuar si:

- Producción no está en v29;
- hard cap aparece activo;
- la copia LAB no es válida;
- existen violaciones de foreign keys;
- el baseline no reconcilia.

Declarar:

PRODUCTION_READ_ONLY = PASS/FAIL
LAB_DATABASE_ISOLATED = PASS/FAIL

==================================================
3. DECISIÓN DE DISEÑO DEL LIFECYCLE
==================================================

Auditar el CHECK actual de:

paper_allocation_request.allocation_status

SQLite no permite ampliar de forma trivial un CHECK existente.

Preferencia:

- evitar reconstruir destructivamente paper_allocation_request;
- conservar las 82 solicitudes históricas intactas;
- añadir tablas append-only de lifecycle y reservas;
- derivar el estado operativo actual desde el último evento válido.

Crear, si resulta compatible con el modelo:

A. paper_allocation_request_event

Responsabilidad:

- lifecycle append-only;
- confirmaciones;
- cancelaciones;
- fallos;
- cambios de estado;
- auditoría.

B. paper_allocation_reservation

Responsabilidad:

- reserva monetaria;
- consumo futuro;
- liberación;
- protección de capacidad.

C. paper_allocation_policy_activation

Responsabilidad:

- auditoría futura de activación;
- en Gate 2F-A1 permanece vacía.

No duplicar sin necesidad el estado económico en múltiples tablas.

Si se decide reconstruir paper_allocation_request para ampliar su CHECK:

- justificarlo;
- demostrar copia íntegra de las 82 filas;
- demostrar preservación de IDs, hashes y foreign keys;
- probar rollback completo;
- no aplicarlo en Producción durante este gate.

Entregar antes de implementar:

LIFECYCLE_STORAGE_DESIGN =
APPEND_ONLY_COMPANION_TABLES /
REQUEST_TABLE_REBUILD

RATIONALE =
<explicación>

La opción preferida es:

APPEND_ONLY_COMPANION_TABLES

==================================================
4. SCHEMA v30 PROPUESTO
==================================================

El schema mínimo debe soportar:

A. REQUEST LIFECYCLE

Campos mínimos:

- request_event_id;
- allocation_request_id;
- previous_status;
- new_status;
- event_type;
- event_at_utc;
- actor;
- effective_session nullable;
- requested_notional;
- proposed_allocated_notional;
- unallocated_notional;
- available_capacity_observed;
- preview_hash nullable;
- confirmation_reference nullable;
- failure_code nullable;
- failure_reason nullable;
- idempotency_key;
- content_hash.

B. RESERVATION

Campos mínimos:

- reservation_id;
- allocation_request_id;
- paper_portfolio_id;
- amount;
- currency;
- status;
- reserved_at_utc;
- effective_session;
- consumed_at_utc nullable;
- released_at_utc nullable;
- invalidated_at_utc nullable;
- release_reason nullable;
- idempotency_key;
- content_hash.

Estados:

ACTIVE
CONSUMED
RELEASED
EXPIRED
INVALIDATED

C. POLICY ACTIVATION AUDIT

Campos mínimos:

- activation_id;
- allocation_policy_id;
- previous_status;
- new_status;
- activated_at_utc;
- effective_from_session;
- approved_by;
- activation_reference;
- policy_version;
- content_hash.

La tabla se crea, pero permanece vacía en Gate 2F-A1.

==================================================
5. ESTADOS DEL LIFECYCLE
==================================================

Estados prospectivos mínimos:

- DRAFT;
- AWAITING_CONFIRMATION;
- PENDING_CAPITAL;
- PENDING_PARTIAL_CONFIRMATION;
- RESERVED_FOR_EXECUTION;
- PARTIALLY_RESERVED_FOR_EXECUTION;
- PENDING_MARKET_CLOSE;
- PENDING_PRICE;
- FULLY_ALLOCATED;
- PARTIALLY_ALLOCATED;
- CANCELLED;
- REJECTED;
- SUPERSEDED;
- FAILED.

Gate 2F-A1 solo debe producir:

- DRAFT;
- AWAITING_CONFIRMATION;
- PENDING_CAPITAL;
- PENDING_PARTIAL_CONFIRMATION;
- RESERVED_FOR_EXECUTION;
- PARTIALLY_RESERVED_FOR_EXECUTION;
- CANCELLED;
- REJECTED;
- SUPERSEDED;
- FAILED.

No producir todavía:

- PENDING_MARKET_CLOSE;
- PENDING_PRICE;
- FULLY_ALLOCATED;
- PARTIALLY_ALLOCATED.

Esos estados corresponden a ejecución económica posterior.

Definir una matriz explícita de transiciones permitidas.

Toda transición no autorizada debe fallar.

==================================================
6. FUENTE CANÓNICA DE PAPER CASH
==================================================

Para una cartera existente:

gross_paper_cash =
paper_cash del último paper_track_snapshot canónico.

El snapshot debe pertenecer a:

- metodología PAPER_PRICE_RETURN_TWR_V1;
- reconstrucción económica canónica;
- última sesión certificada disponible.

No utilizar:

- configured_capital;
- NAV;
- capital menos requests históricos;
- allocated_notional legacy;
- holdings legacy;
- cash real;
- cash de otra cartera.

Si no existe snapshot canónico:

CAPACITY_STATUS =
UNAVAILABLE

No asumir Paper Cash igual al capital configurado.

Para una futura cartera nueva, la capacidad inicial se resolverá en Gate
2F-A2/B después de confirmar su aportación inicial.

==================================================
7. RESERVAS Y CAPACIDAD DISPONIBLE
==================================================

Definición:

active_reserved_cash =
suma de reservations ACTIVE de la cartera.

available_capacity =
max(0, gross_paper_cash - active_reserved_cash)

No restar:

- reservas RELEASED;
- reservas CONSUMED;
- reservas EXPIRED;
- reservas INVALIDATED.

No sumar:

- aportaciones no ejecutadas;
- flujos propuestos;
- NAV no realizado;
- proceeds de ventas propuestas;
- capital real.

Toda respuesta de capacidad debe incluir:

- portfolio_id;
- snapshot_session;
- gross_paper_cash;
- active_reserved_cash;
- available_capacity;
- currency;
- calculation_timestamp;
- active_reservation_count;
- content hash.

==================================================
8. PREVIEW
==================================================

Implementar un servicio interno puro:

previewPaperAllocation

Entrada:

- allocation_request_id o datos candidatos;
- portfolio_id;
- requested_notional;
- optional proposed_partial_notional;
- calculation timestamp.

Salida:

- requested_notional;
- gross_paper_cash;
- active_reserved_cash;
- available_capacity;
- proposed_allocated_notional;
- unallocated_notional;
- expected_status;
- partial_confirmation_required;
- snapshot_session;
- warnings;
- preview_hash.

El preview:

- no escribe;
- no reserva;
- no confirma;
- no crea lifecycle event;
- no crea posición;
- no crea membership;
- no cambia el Track.

El preview puede quedar obsoleto antes de confirmar.

La confirmación siempre debe recalcular capacidad dentro de la transacción.

==================================================
9. CONFIRMACIÓN COMPLETA
==================================================

Implementar un servicio interno:

confirmPaperAllocation

Entrada:

- allocation_request_id;
- portfolio_id;
- preview_hash;
- actor;
- idempotency_key;
- confirmation reference.

Dentro de una única transacción SQLite:

1. cargar request;
2. validar estado;
3. validar requested_notional;
4. cargar último snapshot canónico;
5. calcular gross_paper_cash;
6. sumar reservas ACTIVE;
7. recalcular available_capacity;
8. comprobar idempotency key;
9. decidir estado;
10. crear lifecycle event;
11. crear reserva cuando corresponda;
12. validar capacidad final;
13. COMMIT.

Si:

available_capacity >= requested_notional

crear:

reservation.amount =
requested_notional

reservation.status =
ACTIVE

new_status =
RESERVED_FOR_EXECUTION

Si:

available_capacity < requested_notional

no crear reserva y establecer:

new_status =
PENDING_CAPITAL

La confirmación no crea ninguna operación económica.

==================================================
10. CONFIRMACIÓN PARCIAL
==================================================

Una asignación parcial requiere dos acciones diferenciadas:

1. preview parcial;
2. confirmación parcial explícita.

Condiciones:

- requested_notional > 0;
- proposed_partial_notional > 0;
- proposed_partial_notional < requested_notional;
- proposed_partial_notional <= available_capacity;
- partial policy = REQUIRE_EXPLICIT_CONFIRMATION.

Al confirmar:

reservation.amount =
proposed_partial_notional

unallocated_notional =
requested_notional - proposed_partial_notional

new_status =
PARTIALLY_RESERVED_FOR_EXECUTION

No utilizar automáticamente todo el cash disponible si Omar confirmó un
importe inferior.

No aumentar la reserva posteriormente sin una nueva confirmación.

==================================================
11. CANCELACIÓN
==================================================

Implementar:

cancelPaperAllocation

Debe funcionar sobre estados cancelables.

Dentro de una única transacción:

1. cargar request y estado actual;
2. comprobar idempotencia;
3. localizar reserva ACTIVE;
4. marcar reserva RELEASED;
5. registrar released_at;
6. registrar release_reason;
7. crear lifecycle event CANCELLED;
8. comprobar capacidad liberada;
9. COMMIT.

No borrar la reserva.

No borrar el request.

No borrar eventos.

Una segunda cancelación debe:

- devolver ALREADY_CANCELLED;
- no crear otro evento económico;
- no liberar dos veces.

==================================================
12. EXPIRACIÓN E INVALIDACIÓN
==================================================

Gate 2F-A1 debe modelar, aunque no active automáticamente:

EXPIRED

para una reserva cuyo contrato temporal haya vencido.

INVALIDATED

para una reserva que ya no sea segura por:

- cambio de configuración;
- inconsistencia detectada;
- invalidación administrativa;
- supersesión.

No implementar expiración basada en un cron arbitrario sin contrato.

Por defecto:

expires_at = NULL

hasta que Gate 2F-A2 defina el lifecycle de ejecución EOD.

==================================================
13. CONCURRENCIA
==================================================

La reserva debe ser atómica y race-safe.

Usar:

BEGIN IMMEDIATE

o el mecanismo transaccional equivalente del stack existente.

Caso obligatorio:

Paper Cash =
1.000

Dos solicitudes concurrentes:

A requested =
700

B requested =
700

Resultado permitido:

- una reserva ACTIVE de 700;
- la otra solicitud queda PENDING_CAPITAL;
- available capacity final = 300.

Resultado prohibido:

- dos reservas de 700;
- reserved cash = 1.400;
- available capacity negativo.

Definir orden determinista por:

- adquisición transaccional;
- confirmed_at;
- request ID como desempate cuando sea necesario.

No confiar únicamente en un cálculo realizado antes de iniciar la transacción.

==================================================
14. IDEMPOTENCIA
==================================================

Claves de idempotencia obligatorias para:

- creación del request prospectivo en LAB;
- confirmación;
- confirmación parcial;
- cancelación;
- reserva;
- lifecycle event.

Probar:

- doble clic;
- retry HTTP futuro;
- reinicio después de COMMIT;
- reinicio antes de COMMIT;
- mismo idempotency key con payload igual;
- mismo idempotency key con payload distinto.

Reglas:

Mismo key + mismo payload:

- devolver el resultado original;
- no duplicar.

Mismo key + payload distinto:

- IDEMPOTENCY_CONFLICT;
- no escribir.

==================================================
15. IDENTIDAD DE CAPACIDAD
==================================================

Después de cada operación:

active_reserved_cash <= gross_paper_cash

available_capacity >= 0

gross_paper_cash =
active_reserved_cash
+ available_capacity

sujeto únicamente a precisión monetaria documentada.

No utilizar epsilon para ocultar un sobregiro material.

Preferir almacenamiento monetario en minor units si es compatible con el
patrón del repositorio.

Si v29 utiliza REAL:

- documentar precisión;
- normalizar entradas;
- definir MONEY_EPSILON;
- comprobar invariantes antes de COMMIT.

==================================================
16. REQUESTS HISTÓRICOS
==================================================

Las 82 solicitudes históricas permanecen intactas.

No crear lifecycle prospectivo retroactivo para ellas salvo un evento
explícito de importación que:

- no cambie su estado histórico;
- no cree reservas;
- no las haga ejecutables;
- no active la política.

Preferencia para Gate 2F-A1:

HISTORICAL_REQUESTS_MIGRATED_TO_NEW_LIFECYCLE = 0

El lifecycle nuevo se aplica únicamente a requests prospectivos creados en
LAB después de la migración.

==================================================
17. POSICIÓN 11
==================================================

Caso obligatorio:

Cartera:

- configured capital = 10.000;
- Paper Cash canónico = 0;
- diez posiciones existentes;
- requested_notional nuevo = 1.000.

Resultado:

preview.expected_status =
PENDING_CAPITAL

confirmación:

- no crea reserva;
- status = PENDING_CAPITAL;
- available capacity = 0;
- posiciones existentes intactas;
- cero composición;
- cero transacción;
- cero snapshot;
- cero rebalanceo.

Declarar:

POSITION_11_PENDING = PASS/FAIL

==================================================
18. MIGRACIÓN v30 EN LAB
==================================================

Crear migración v29 → v30 mediante el runner existente.

Orden:

1. BEGIN IMMEDIATE;
2. comprobar user_version = 29;
3. crear tablas nuevas;
4. crear índices;
5. crear constraints;
6. validar schema;
7. establecer user_version = 30;
8. COMMIT.

Si falla:

- ROLLBACK;
- mantener v29;
- no dejar estructuras parciales.

Probar:

A. migración inicial;
B. segunda ejecución;
C. foreign_key_check;
D. integrity_check;
E. rollback vacío v30 → v29;
F. reaplicar v30;
G. 82 requests históricos intactos;
H. todas las tablas económicas existentes intactas.

No aplicar a Producción.

==================================================
19. ÍNDICES Y CONSTRAINTS
==================================================

Definir al menos:

A. Unicidad de idempotency keys.

B. Una sola reserva ACTIVE por allocation request.

Como SQLite no admite necesariamente un CHECK transversal, utilizar:

- índice único parcial;
- transacción;
- validación de servicio.

Ejemplo conceptual:

UNIQUE allocation_request_id
WHERE status = 'ACTIVE'

C. amount > 0.

D. timestamps UTC.

E. effective_session como fecha NYSE nullable.

F. reservation currency coherente con la cartera.

G. lifecycle event append-only.

No utilizar ON DELETE CASCADE que pueda eliminar la auditoría financiera.

Preferir RESTRICT o NO ACTION.

==================================================
20. AUDITORÍA
==================================================

Cada evento debe conservar:

- allocation_request_id;
- actor;
- timestamp UTC;
- old status;
- new status;
- requested_notional;
- proposed allocated notional;
- available capacity observada;
- preview hash;
- confirmation reference;
- idempotency key;
- content hash.

Cada reserva:

- referencia al request;
- referencia a cartera;
- importe;
- estado;
- sesión prevista;
- timestamps;
- content hash.

No guardar secretos, tokens ni payloads innecesarios.

==================================================
21. SERVICIOS INTERNOS
==================================================

Crear servicios desacoplados de HTTP y UI:

- calculatePaperAvailableCapacity;
- previewPaperAllocation;
- confirmPaperAllocation;
- confirmPartialPaperAllocation;
- cancelPaperAllocation;
- getPaperAllocationLifecycle;
- getActivePaperReservations.

Estos servicios deben poder probarse directamente en LAB.

No crear todavía rutas HTTP públicas.

No integrar todavía con la pantalla de tesis.

No integrar todavía con daily-close.

==================================================
22. PRUEBAS
==================================================

Crear:

verify-paper-v30-migration.mjs
verify-paper-request-lifecycle.mjs
verify-paper-capacity-calculation.mjs
verify-paper-capacity-reservation.mjs
verify-paper-full-confirmation.mjs
verify-paper-partial-confirmation.mjs
verify-paper-reservation-cancellation.mjs
verify-paper-reservation-idempotency.mjs
verify-paper-reservation-concurrency.mjs
verify-paper-hard-cap-position-11.mjs
verify-paper-hard-cap-isolation.mjs

Casos mínimos:

A. Migración v29 → v30.
B. Rollback vacío.
C. Requests históricos intactos.
D. Preview no escribe.
E. Cash suficiente reserva completo.
F. Cash insuficiente queda pending.
G. Parcial exige confirmación.
H. Parcial conserva requested.
I. Cancelación libera reserva.
J. Doble cancelación idempotente.
K. Dos requests compiten por cash.
L. Doble confirmación no duplica.
M. Payload distinto con mismo key falla.
N. Capacidad nunca negativa.
O. Posición 11 queda pending.
P. Cero composiciones creadas.
Q. Cero transacciones creadas.
R. Cero snapshots creados.
S. Política inactiva.
T. Producción intacta.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- suites v27/v28/v29;
- Gate 2D-2;
- Gate 2E-A;
- financial core;
- repository integrity.

==================================================
23. AISLAMIENTO
==================================================

Después de las pruebas LAB verificar Producción:

- schema sigue v29;
- integrity_check;
- foreign_key_check;
- hard cap sigue sin effective_from;
- snapshots Papel idénticos;
- motor Papel idéntico;
- gráficos idénticos;
- holdings legacy idénticos;
- Book real idéntico;
- cash real idéntico;
- NAV real idéntico;
- tesis y catalizadores idénticos.

Resultados:

PRODUCTION_SCHEMA_UNCHANGED = PASS
PRODUCTION_DATA_UNCHANGED = PASS
PAPER_TRACK_UNCHANGED = PASS
REAL_BOOK_UNCHANGED = PASS
REAL_CASH_UNCHANGED = PASS
REAL_NAV_UNCHANGED = PASS
NO_POLICY_ACTIVATED = PASS

==================================================
24. CÓDIGO Y COMMIT
==================================================

Añadir únicamente:

- migración v30;
- rollback vacío;
- lifecycle append-only;
- reservations;
- capacity calculator;
- preview service;
- confirmation service;
- partial confirmation service;
- cancellation service;
- concurrency protection;
- pruebas;
- documentación técnica.

No añadir:

- rutas HTTP;
- UI;
- daily-close integration;
- catch-up integration;
- compras;
- ventas;
- composiciones prospectivas;
- memberships prospectivos;
- carteras nuevas;
- aportaciones;
- hard-cap activation;
- scratchpad;
- backups;
- bases;
- secretos.

Ejecutar secret scan.

Crear un único commit local:

feat(lens): add atomic paper allocation reservations

No crear tag.
No hacer push.

==================================================
25. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = 0466595
CANONICAL_HEAD_AFTER = <hash>

PRODUCTION_SCHEMA_BEFORE = v29
PRODUCTION_SCHEMA_AFTER = v29

LAB_SCHEMA_BEFORE = v29
LAB_SCHEMA_AFTER = v30

LIFECYCLE_STORAGE_DESIGN =
<diseño>

V30_TABLES_CREATED = <lista>
V30_INDEXES_CREATED = <número>
V30_FOREIGN_KEYS_CREATED = <número>

HISTORICAL_REQUESTS_PRESERVED = <número>/82
HISTORICAL_REQUESTS_MIGRATED_TO_NEW_LIFECYCLE = 0

REQUEST_LIFECYCLE = PASS/FAIL
CAPACITY_CALCULATION = PASS/FAIL
CAPACITY_RESERVATION = PASS/FAIL
CAPACITY_RACE_PROTECTION = PASS/FAIL

FULL_CONFIRMATION = PASS/FAIL
PENDING_CAPITAL = PASS/FAIL
PARTIAL_CONFIRMATION = PASS/FAIL
PARTIAL_CONFIRMATION_REQUIRED = PASS/FAIL
CANCELLATION_RELEASES_RESERVATION = PASS/FAIL

REQUEST_IDEMPOTENCY = PASS/FAIL
RESERVATION_IDEMPOTENCY = PASS/FAIL
CANCELLATION_IDEMPOTENCY = PASS/FAIL
IDEMPOTENCY_PAYLOAD_CONFLICT = PASS/FAIL

POSITION_11_PENDING = PASS/FAIL
AVAILABLE_CAPACITY_NEVER_NEGATIVE = PASS/FAIL
ACTIVE_RESERVATIONS_NEVER_EXCEED_CASH = PASS/FAIL

PROSPECTIVE_COMPOSITIONS_CREATED = 0
PROSPECTIVE_TRANSACTIONS_CREATED = 0
PROSPECTIVE_MEMBERSHIPS_CREATED = 0
PAPER_TRACK_SNAPSHOTS_CREATED = 0

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

PRODUCTION_SCHEMA_UNCHANGED = PASS/FAIL
PRODUCTION_DATA_UNCHANGED = PASS/FAIL
PAPER_TRACK_UNCHANGED = PASS/FAIL
REAL_BOOK_UNCHANGED = PASS/FAIL
REAL_CASH_UNCHANGED = PASS/FAIL
REAL_NAV_UNCHANGED = PASS/FAIL
NO_POLICY_ACTIVATED = PASS/FAIL
NO_REAL_ECONOMIC_WRITES = PASS/FAIL

SECRET_SCAN = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

LOCAL_COMMIT = <hash>

EXACT_BLOCKERS_BEFORE_GATE_2F_A2:
1. <bloqueo>
2. <bloqueo>
3. <bloqueo>

Detenerse después de Gate 2F-A1.