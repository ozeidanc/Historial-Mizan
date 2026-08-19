MIZAN · LA LENTE
GATE 2F-A1 · FUNDACIÓN TRANSACCIONAL DEL HARD CAP
NUEVAS TESIS + ALERTAS DE CATALIZADOR + RESERVAS DE CASH Y CANTIDAD
SCHEMA v30 · LIFECYCLE · CONCURRENCIA · LAB ÚNICAMENTE

NATURALEZA

Esta ejecución autoriza exclusivamente:

1. diseñar la ampliación mínima del schema prospectivo;
2. crear una migración v29 → v30;
3. probar la migración únicamente en LAB;
4. implementar el lifecycle append-only de solicitudes prospectivas;
5. implementar el cálculo canónico de Paper Cash disponible;
6. implementar reservas atómicas de Paper Cash para futuras compras;
7. implementar reservas atómicas de cantidad para reducciones y ventas;
8. implementar confirmación completa de una nueva asignación;
9. implementar confirmación parcial de una nueva asignación;
10. implementar decisiones de mantener, reducir o vender ante alertas de
    catalizador;
11. implementar cancelación y liberación de reservas;
12. implementar protección contra concurrencia, doble confirmación y doble
    venta;
13. implementar servicios internos puros;
14. crear pruebas económicas y de aislamiento;
15. crear un único commit local si todo queda verde.

No autoriza:

- aplicar v30 en Producción;
- activar MANUAL_NOTIONAL_HARD_CAP;
- fijar effective_from;
- ejecutar compras;
- ejecutar ventas;
- ejecutar reducciones;
- crear composiciones prospectivas;
- crear memberships prospectivos;
- crear posiciones globales prospectivas;
- crear snapshots nuevos;
- modificar daily-close;
- modificar startup catch-up;
- crear endpoints HTTP públicos de escritura;
- modificar UI productiva;
- crear carteras nuevas;
- crear aportaciones externas;
- crear retiradas externas;
- crear reasignaciones prospectivas;
- cerrar memberships;
- modificar alertas o estados de catalizador existentes;
- modificar tesis;
- modificar el Track Papel;
- modificar sus gráficas;
- modificar datos históricos;
- modificar Book, cash o NAV reales;
- modificar holdings o snapshots legacy;
- desplegar;
- hacer push;
- crear tag;
- realizar operaciones Git remotas.

Toda escritura de prueba debe realizarse exclusivamente en una copia LAB
aislada.

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

HISTORICAL_TRACK_POLICY =
AS_OPERATED

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

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

EXIT_POLICY =
KEEP_AS_PAPER_CASH

La aplicación muestra actualmente, entre otros datos:

Capital objetivo =
50.000,00

Capital efectivo asignado =
50.000,14

Capital no asignado =
0,00

Asignado =
100 %

Estos datos agregados no prueban la existencia de Paper Cash disponible.

Para hard cap, la capacidad se determinará exclusivamente por cartera mediante
el último estado económico canónico y sus reservas activas.

==================================================
1. ALERTAS DE CATALIZADOR OBSERVADAS
==================================================

Actualmente Mizan muestra alertas confirmadas en dos escaneos consecutivos.

CATALYST_NO_LONGER_ACTIVE:

- ONB;
- ATRC;
- ISRG;
- HCSG;
- WSBC;
- WTFC.

CATALYST_REVERTED:

- PEGA.

Los mensajes indican que Omar debe revisar si mantiene o cierra la posición.

Estas alertas:

- son informativas;
- no constituyen una orden;
- no autorizan una venta;
- no modifican la posición;
- no modifican la tesis;
- no cierran el membership;
- no generan Paper Cash;
- no cambian el Track.

Las categorías mostradas como:

- C1;
- C2;
- C3;
- C4;
- C6;

son categorías del catalizador.

No deben interpretarse automáticamente como:

- cartera papel;
- portfolio_id;
- orden de salida;
- prioridad de venta;
- tamaño de posición.

Antes de ofrecer una acción económica, reconciliar cada ticker con su cartera
papel real y su exposición económica canónica.

==================================================
2. RESULTADO FUNCIONAL DE 2F-A1
==================================================

Gate 2F-A1 debe implementar internamente en LAB dos familias de decisiones.

A. NUEVA TESIS / NUEVA INCORPORACIÓN

DRAFT
→ AWAITING_CONFIRMATION
→ CONFIRMED
→ CAPACITY_CHECK
→ RESERVED_FOR_EXECUTION

o, si no hay Paper Cash:

DRAFT
→ AWAITING_CONFIRMATION
→ CONFIRMED
→ PENDING_CAPITAL

Asignación parcial:

PENDING_CAPITAL
→ PENDING_PARTIAL_CONFIRMATION
→ confirmación explícita
→ PARTIALLY_RESERVED_FOR_EXECUTION

B. REVISIÓN DE POSICIÓN POR CATALIZADOR

ACTION_REQUIRED
→ HOLD_ACKNOWLEDGED

o:

ACTION_REQUIRED
→ REDUCTION_DRAFT
→ AWAITING_CONFIRMATION
→ QUANTITY_RESERVED_FOR_EXECUTION

o:

ACTION_REQUIRED
→ EXIT_DRAFT
→ AWAITING_CONFIRMATION
→ QUANTITY_RESERVED_FOR_EXECUTION

Cancelación:

estado cancelable
→ CANCELLED
→ reserva liberada

Gate 2F-A1 termina en la reserva de cash o cantidad.

No ejecuta compras ni ventas.

No crea exposición.

No modifica el Track.

==================================================
3. PRINCIPIO DE DECISIÓN EXPLÍCITA
==================================================

Una tesis aprobada no crea automáticamente una posición.

Una alerta de catalizador no vende automáticamente una posición.

Mizan debe ofrecer decisiones explícitas.

Para una tesis nueva:

- AÑADIR A CARTERA PAPEL;
- ELEGIR CARTERA;
- INDICAR REQUESTED_NOTIONAL;
- REVISAR PREVIEW;
- CONFIRMAR;
- CANCELAR.

Para una alerta de catalizador:

- MANTENER;
- REDUCIR POSICIÓN;
- VENDER POSICIÓN COMPLETA;
- CANCELAR.

No se considera confirmación:

- abrir el modal;
- seleccionar una cartera;
- escribir un importe;
- abrir la alerta;
- cerrar la alerta;
- cambiar de pestaña;
- aprobar una tesis;
- reconocer que el catalizador cambió;
- recargar la página.

Toda decisión económica exige confirmación explícita e idempotente.

==================================================
4. PRECONDICIONES
==================================================

Antes de modificar código:

- confirmar HEAD 0466595;
- confirmar schema v29;
- confirmar env-info;
- confirmar una única raíz Git;
- inventariar archivos untracked;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- confirmar health;
- confirmar una sola instancia;
- confirmar política hard cap inactiva;
- capturar contadores e hashes productivos;
- crear una copia LAB validada;
- ejecutar integrity_check sobre LAB;
- ejecutar foreign_key_check sobre LAB.

Capturar baseline de:

- paper_allocation_policy;
- paper_portfolio_track_config;
- paper_allocation_request;
- paper_portfolio_flow;
- paper_portfolio_transaction;
- paper_composition_version;
- paper_composition_member;
- paper_global_position;
- paper_portfolio_membership_episode;
- paper_track_snapshot;
- paper_economic_*;
- alertas de catalizador;
- estados de catalizador;
- tesis;
- carteras_papel;
- holdings legacy;
- Track Papel;
- Book, cash y NAV reales.

No continuar si:

- Producción no está en v29;
- hard cap aparece activo;
- la copia LAB no es válida;
- existen violaciones de foreign keys;
- el baseline no reconcilia;
- las posiciones activas no pueden vincularse con sus memberships económicos.

Declarar:

PRODUCTION_READ_ONLY = PASS/FAIL
LAB_DATABASE_ISOLATED = PASS/FAIL

==================================================
5. MODELO UNIFICADO DE SOLICITUDES
==================================================

Modelar dos tipos principales de solicitud prospectiva.

A. ALLOCATION

Para:

- añadir una nueva tesis;
- ampliar una posición en el futuro;
- reservar Paper Cash.

B. POSITION_ACTION

Para:

- mantener;
- reducir;
- vender completamente;
- reservar cantidad.

Tipos mínimos:

- NEW_POSITION;
- INCREASE_POSITION;
- HOLD_POSITION;
- PARTIAL_EXIT;
- FULL_EXIT.

Cada solicitud debe identificar de forma inequívoca:

- request_id;
- request_type;
- paper_portfolio_id;
- global_position_id nullable;
- membership_episode_id nullable;
- thesis_id nullable;
- ticker;
- source_event_type;
- source_event_id nullable;
- source_catalyst_id nullable;
- source_catalyst_status nullable;
- requested_by;
- requested_at_utc;
- current_status;
- idempotency_key;
- content_hash.

No utilizar solamente ticker como identidad.

==================================================
6. DECISIÓN DE DISEÑO DEL LIFECYCLE
==================================================

Auditar el CHECK actual de:

paper_allocation_request.allocation_status

SQLite no permite ampliar de forma trivial un CHECK existente.

Preferencia:

- no reconstruir destructivamente paper_allocation_request;
- conservar las 82 solicitudes históricas intactas;
- añadir tablas append-only de lifecycle, decisiones y reservas;
- derivar el estado operativo desde el último evento válido.

Diseño preferido:

A. paper_allocation_request_event

Responsabilidad:

- lifecycle append-only de nuevas incorporaciones;
- confirmaciones;
- cancelaciones;
- fallos;
- cambios de estado;
- auditoría.

B. paper_position_action_request

Responsabilidad:

- solicitud de mantener, reducir o vender;
- vínculo con posición, membership y catalizador;
- cantidad solicitada;
- estado actual.

C. paper_position_action_event

Responsabilidad:

- lifecycle append-only de las acciones de posición.

D. paper_allocation_reservation

Responsabilidad:

- reserva monetaria;
- consumo futuro;
- liberación;
- protección de capacidad.

E. paper_quantity_reservation

Responsabilidad:

- reserva de cantidad;
- evitar doble venta;
- consumo o liberación futuros.

F. paper_allocation_policy_activation

Responsabilidad:

- auditoría futura de activación;
- debe permanecer vacía en Gate 2F-A1.

Entregar antes de implementar:

LIFECYCLE_STORAGE_DESIGN =
APPEND_ONLY_COMPANION_TABLES /
REQUEST_TABLE_REBUILD

RATIONALE =
<explicación>

La opción preferida es:

APPEND_ONLY_COMPANION_TABLES

Si se decide reconstruir una tabla existente:

- justificarlo;
- demostrar copia íntegra;
- preservar IDs;
- preservar hashes;
- preservar foreign keys;
- probar rollback completo;
- no aplicarlo en Producción.

==================================================
7. SCHEMA v30 PROPUESTO
==================================================

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

B. CASH RESERVATION

Campos mínimos:

- reservation_id;
- allocation_request_id;
- paper_portfolio_id;
- amount;
- currency;
- status;
- reserved_at_utc;
- effective_session nullable;
- consumed_at_utc nullable;
- released_at_utc nullable;
- invalidated_at_utc nullable;
- release_reason nullable;
- idempotency_key;
- content_hash.

Estados:

- ACTIVE;
- CONSUMED;
- RELEASED;
- EXPIRED;
- INVALIDATED.

C. POSITION ACTION REQUEST

Campos mínimos:

- position_action_request_id;
- action_type;
- paper_portfolio_id;
- global_position_id;
- membership_episode_id;
- ticker;
- catalyst_reference nullable;
- catalyst_status_observed nullable;
- scan_confirmation_reference nullable;
- requested_reduction_basis nullable;
- requested_reduction_value nullable;
- canonical_quantity_observed;
- estimated_notional nullable;
- estimate_session nullable;
- status;
- requested_by;
- requested_at_utc;
- idempotency_key;
- content_hash.

D. POSITION ACTION EVENT

Campos mínimos:

- position_action_event_id;
- position_action_request_id;
- previous_status;
- new_status;
- event_type;
- event_at_utc;
- actor;
- canonical_quantity_observed;
- available_quantity_observed;
- proposed_reserved_quantity;
- estimated_notional nullable;
- preview_hash nullable;
- confirmation_reference nullable;
- rationale nullable;
- review_again_at nullable;
- failure_code nullable;
- failure_reason nullable;
- idempotency_key;
- content_hash.

E. QUANTITY RESERVATION

Campos mínimos:

- quantity_reservation_id;
- position_action_request_id;
- paper_portfolio_id;
- global_position_id;
- membership_episode_id;
- quantity;
- status;
- reserved_at_utc;
- effective_session nullable;
- consumed_at_utc nullable;
- released_at_utc nullable;
- invalidated_at_utc nullable;
- release_reason nullable;
- idempotency_key;
- content_hash.

Estados:

- ACTIVE;
- CONSUMED;
- RELEASED;
- EXPIRED;
- INVALIDATED.

F. POLICY ACTIVATION AUDIT

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

Crear la tabla, pero mantenerla vacía.

==================================================
8. ESTADOS DEL LIFECYCLE
==================================================

A. NUEVAS ASIGNACIONES

Estados mínimos:

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

Gate 2F-A1 puede producir únicamente:

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

B. ACCIONES SOBRE POSICIONES

Estados mínimos:

- ACTION_REQUIRED;
- HOLD_ACKNOWLEDGED;
- REDUCTION_DRAFT;
- EXIT_DRAFT;
- AWAITING_CONFIRMATION;
- QUANTITY_RESERVED_FOR_EXECUTION;
- PENDING_MARKET_CLOSE;
- PENDING_PRICE;
- PARTIALLY_EXECUTED;
- EXECUTED;
- CANCELLED;
- REJECTED;
- SUPERSEDED;
- FAILED.

Gate 2F-A1 puede producir únicamente:

- ACTION_REQUIRED;
- HOLD_ACKNOWLEDGED;
- REDUCTION_DRAFT;
- EXIT_DRAFT;
- AWAITING_CONFIRMATION;
- QUANTITY_RESERVED_FOR_EXECUTION;
- CANCELLED;
- REJECTED;
- SUPERSEDED;
- FAILED.

No producir todavía:

- PENDING_MARKET_CLOSE;
- PENDING_PRICE;
- PARTIALLY_EXECUTED;
- EXECUTED.

Crear matrices explícitas de transiciones.

Toda transición no permitida debe fallar.

==================================================
9. ELEGIBILIDAD PARA REDUCIR O VENDER
==================================================

Mostrar o permitir REDUCIR/VENDER únicamente cuando exista:

- paper_global_position identificada;
- membership económico abierto;
- cartera papel identificada;
- cantidad económica canónica > 0;
- composición económica vigente;
- último snapshot canónico;
- lineage resuelto.

No permitir REDUCIR/VENDER cuando:

- el ticker no pertenece a una cartera papel;
- solo existe membership de auditoría sin exposición;
- la posición ya fue vendida;
- la cantidad económica es cero;
- existe una reserva activa de venta total;
- no puede resolverse la cartera;
- el estado económico está bloqueado.

Si un ticker aparece en varias carteras:

- mostrar cada cartera por separado;
- mostrar cantidad por cartera;
- mostrar valor estimado por cartera;
- exigir seleccionar una cartera;
- no crear una venta global implícita.

No inferir la cartera desde C1, C2, C3, C4 o C6.

==================================================
10. FUENTE CANÓNICA DE PAPER CASH
==================================================

Por cartera:

gross_paper_cash =
paper_cash del último paper_track_snapshot canónico.

El snapshot debe pertenecer a:

- metodología PAPER_PRICE_RETURN_TWR_V1;
- reconstrucción económica canónica;
- última sesión certificada.

No utilizar:

- Capital objetivo agregado;
- Capital efectivo asignado agregado;
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

==================================================
11. FUENTE CANÓNICA DE CANTIDAD
==================================================

Por posición y membership:

canonical_quantity =
cantidad económica de la última composición canónica vigente.

No utilizar:

- cantidad legacy;
- cantidad original;
- cantidad solicitada;
- cantidad inferida desde P&L;
- cantidad de otra cartera;
- cantidad derivada de precio live.

Definir:

active_reserved_quantity =
suma de quantity reservations ACTIVE
para esa posición, membership y cartera.

available_quantity =
max(0, canonical_quantity - active_reserved_quantity)

No contar reservas:

- RELEASED;
- CONSUMED;
- EXPIRED;
- INVALIDATED.

Toda respuesta debe indicar:

- paper_portfolio_id;
- global_position_id;
- membership_episode_id;
- canonical_quantity;
- active_reserved_quantity;
- available_quantity;
- snapshot_session;
- price_reference;
- calculation_timestamp;
- content_hash.

==================================================
12. CAPACIDAD DISPONIBLE PARA COMPRAS
==================================================

Definición:

active_reserved_cash =
suma de cash reservations ACTIVE de la cartera.

available_capacity =
max(0, gross_paper_cash - active_reserved_cash)

No restar:

- reservas RELEASED;
- reservas CONSUMED;
- reservas EXPIRED;
- reservas INVALIDATED.

No sumar:

- aportaciones propuestas;
- NAV;
- proceeds de ventas no ejecutadas;
- estimaciones de ventas;
- capital real;
- Capital objetivo agregado.

Importante:

Una reducción o venta confirmada pero todavía no ejecutada no crea Paper Cash
disponible.

El cash solo existe después de la ejecución EOD de la venta.

==================================================
13. PREVIEW DE NUEVA ASIGNACIÓN
==================================================

Crear un servicio puro:

previewPaperAllocation

Entrada:

- allocation_request_id o datos candidatos;
- paper_portfolio_id;
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
- no crea eventos;
- no crea posición;
- no crea membership;
- no cambia el Track.

La confirmación debe recalcular la capacidad transaccionalmente.

==================================================
14. PREVIEW DE REDUCCIÓN O VENTA
==================================================

Crear un servicio puro:

previewPaperPositionAction

Entrada:

- paper_portfolio_id;
- global_position_id;
- membership_episode_id;
- action_type;
- reduction basis;
- reduction value;
- calculation timestamp.

Tipos:

- HOLD_POSITION;
- PARTIAL_EXIT;
- FULL_EXIT.

Para PARTIAL_EXIT permitir:

A. PERCENTAGE

0 < percentage < 100

B. QUANTITY

0 < quantity < available_quantity

Para FULL_EXIT:

proposed_reserved_quantity =
available_quantity

Salida:

- ticker;
- cartera;
- action type;
- canonical quantity;
- active reserved quantity;
- available quantity;
- proposed reserved quantity;
- remaining quantity;
- last certified close;
- estimate session;
- estimated proceeds;
- Paper Cash actual;
- estimated Paper Cash after execution;
- catalyst status;
- warnings;
- preview hash.

El notional mostrado es una estimación:

estimated_proceeds =
proposed_reserved_quantity
× last certified close

Debe etiquetarse:

ESTIMACIÓN, NO PRECIO DE EJECUCIÓN.

El preview no crea Paper Cash.

No cambia el Track.

No cierra el membership.

==================================================
15. OPCIÓN MANTENER
==================================================

Implementar un servicio interno:

acknowledgeHoldPosition

Resultado:

decision status =
HOLD_ACKNOWLEDGED

Registrar:

- posición;
- membership;
- cartera;
- catalizador;
- estado observado;
- referencia a los dos escaneos;
- decided_by;
- decided_at_utc;
- rationale opcional;
- review_again_at nullable;
- idempotency key;
- content hash.

MANTENER:

- no crea reserva;
- no crea transacción;
- no cambia cantidad;
- no cambia Paper Cash;
- no modifica la tesis;
- no reactiva el catalizador;
- no impide nuevas alertas si aparece evidencia posterior.

==================================================
16. CONFIRMACIÓN COMPLETA DE COMPRA
==================================================

Crear:

confirmPaperAllocation

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
11. crear cash reservation cuando corresponda;
12. comprobar invariantes;
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

no crear reserva.

new_status =
PENDING_CAPITAL

La confirmación no crea una compra.

==================================================
17. CONFIRMACIÓN PARCIAL DE COMPRA
==================================================

Requiere:

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

No utilizar automáticamente todo el cash disponible.

No ampliar después la reserva sin una nueva confirmación.

==================================================
18. CONFIRMACIÓN DE REDUCCIÓN
==================================================

Crear:

confirmPaperPositionReduction

Dentro de una única transacción:

1. cargar position action request;
2. validar PARTIAL_EXIT;
3. validar estado;
4. cargar membership económico abierto;
5. cargar canonical quantity;
6. sumar quantity reservations ACTIVE;
7. recalcular available quantity;
8. validar preview hash;
9. validar idempotency key;
10. comprobar proposed quantity;
11. crear quantity reservation ACTIVE;
12. crear lifecycle event;
13. comprobar invariantes;
14. COMMIT.

Debe cumplirse:

0 < reserved_quantity < available_quantity_before

La reducción no se ejecuta en Gate 2F-A1.

No cambia:

- cantidad económica;
- composición;
- Paper Cash;
- Paper NAV;
- Paper Units;
- Track.

==================================================
19. CONFIRMACIÓN DE VENTA TOTAL
==================================================

Crear:

confirmPaperFullExit

Dentro de una única transacción:

1. cargar position action request;
2. validar FULL_EXIT;
3. validar membership abierto;
4. cargar canonical quantity;
5. sumar reservas ACTIVE;
6. calcular available quantity;
7. validar preview hash;
8. validar idempotency key;
9. reservar toda la available quantity;
10. crear lifecycle event;
11. comprobar invariantes;
12. COMMIT.

Después de una reserva total ACTIVE:

available_quantity =
0

No permitir otra reducción o venta sobre la misma cantidad.

La posición sigue económicamente activa hasta que la venta se ejecute en un
gate posterior.

No cerrar todavía el membership.

==================================================
20. RESERVA DE CANTIDAD
==================================================

Las reservas de cantidad evitan:

- doble venta;
- vender más de la cantidad disponible;
- reducción y venta total simultáneas;
- dos alertas que generen acciones incompatibles;
- doble clic;
- reintentos duplicados.

Invariantes:

active_reserved_quantity <= canonical_quantity

available_quantity >= 0

canonical_quantity =
active_reserved_quantity + available_quantity

sujeto a precisión documentada.

Si existe una reserva total ACTIVE:

- no crear otra reserva;
- devolver QUANTITY_ALREADY_FULLY_RESERVED.

Si existe una reserva parcial ACTIVE:

- una nueva reducción puede permitirse únicamente sobre available_quantity;
- siempre con preview y confirmación nuevos.

==================================================
21. CANCELACIÓN
==================================================

Implementar:

cancelPaperAllocation

y:

cancelPaperPositionAction

Dentro de una única transacción:

1. cargar request;
2. comprobar estado cancelable;
3. comprobar idempotencia;
4. localizar reserva ACTIVE;
5. marcar reserva RELEASED;
6. registrar released_at;
7. registrar release_reason;
8. crear lifecycle event CANCELLED;
9. comprobar capacidad o cantidad liberada;
10. COMMIT.

No borrar:

- requests;
- events;
- reservations;
- alertas;
- tesis.

Una segunda cancelación:

- devuelve ALREADY_CANCELLED;
- no duplica eventos;
- no libera dos veces.

==================================================
22. EXPIRACIÓN E INVALIDACIÓN
==================================================

Modelar:

EXPIRED

para reservas cuyo contrato temporal haya vencido.

INVALIDATED

para reservas que dejan de ser seguras debido a:

- cambio de configuración;
- membership cerrado por otro proceso;
- cantidad canónica modificada;
- supersesión;
- inconsistencia;
- invalidación administrativa.

En Gate 2F-A1:

expires_at =
NULL

por defecto.

No implementar expiración automática hasta definir el contrato EOD en Gate
2F-A2.

==================================================
23. CONCURRENCIA DE CASH
==================================================

Usar:

BEGIN IMMEDIATE

o mecanismo equivalente.

Caso:

Paper Cash =
1.000

Request A =
700

Request B =
700

Resultado permitido:

- una reserva ACTIVE de 700;
- la otra request queda PENDING_CAPITAL;
- available capacity = 300.

Resultado prohibido:

- dos reservas de 700;
- reserved cash = 1.400;
- capacidad negativa.

No confiar en previews previos a la transacción.

==================================================
24. CONCURRENCIA DE CANTIDAD
==================================================

Caso:

canonical quantity =
100

Request A:
reducir 60

Request B:
vender 100

Si A confirma primero:

- A reserva 60;
- available quantity = 40;
- B no puede reservar 100;
- B debe recibir QUANTITY_CAPACITY_CONFLICT.

Si B confirma primero:

- B reserva 100;
- available quantity = 0;
- A no puede reservar 60.

Resultado prohibido:

active reserved quantity > 100

No permitir que una alerta duplicada genere una segunda venta.

==================================================
25. IDEMPOTENCIA
==================================================

Claves obligatorias para:

- creación de request prospectivo;
- creación de position action request;
- preview referenciado;
- confirmación;
- confirmación parcial;
- confirmación de reducción;
- confirmación de salida total;
- mantener;
- cancelación;
- cash reservation;
- quantity reservation;
- lifecycle event.

Mismo key + mismo payload:

- devolver resultado original;
- no duplicar.

Mismo key + payload diferente:

- IDEMPOTENCY_CONFLICT;
- no escribir.

Probar:

- doble clic;
- retry;
- reinicio antes de COMMIT;
- reinicio después de COMMIT;
- dos confirmaciones simultáneas;
- cancelación simultánea con confirmación.

==================================================
26. REQUESTS HISTÓRICOS
==================================================

Las 82 solicitudes históricas permanecen intactas.

No crear lifecycle prospectivo retroactivo.

No crear reservas retroactivas.

No convertir alertas históricas en órdenes.

Resultado esperado:

HISTORICAL_REQUESTS_MIGRATED_TO_NEW_LIFECYCLE = 0

El lifecycle nuevo solo se aplica a fixtures LAB prospectivos.

==================================================
27. ALERTAS ACTUALES Y LINEAGE
==================================================

En LAB, generar un preview de reconciliación para:

- ONB;
- ATRC;
- ISRG;
- HCSG;
- WSBC;
- WTFC;
- PEGA.

Por ticker entregar:

- catalyst category;
- catalyst state;
- alert source;
- scan confirmation;
- global position match;
- open economic membership;
- actual paper portfolio;
- canonical quantity;
- active quantity reservation;
- action eligibility;
- blockers.

No crear solicitudes productivas para esos tickers.

No reservar sus cantidades en Producción.

Objetivo:

demostrar que el futuro flujo sabrá sobre qué posición y cartera actuar.

Declarar:

CATALYST_ALERTS_RECONCILED = <número>/7
CATALYST_ALERTS_UNRESOLVED = <número>

Si un ticker no puede reconciliarse:

- informar;
- no inventar su cartera;
- no bloquear todo el schema;
- bloquear únicamente las acciones económicas de ese ticker.

==================================================
28. POSICIÓN 11
==================================================

Caso obligatorio:

- configured capital = 10.000;
- Paper Cash canónico = 0;
- diez posiciones existentes;
- requested_notional = 1.000.

Resultado:

preview.expected_status =
PENDING_CAPITAL

Confirmación:

- no crea cash reservation;
- status = PENDING_CAPITAL;
- available capacity = 0;
- posiciones existentes intactas;
- cero rebalanceo;
- cero composición;
- cero transacción;
- cero snapshot.

Declarar:

POSITION_11_PENDING = PASS/FAIL

==================================================
29. RELACIÓN ENTRE VENTAS Y CAPACIDAD
==================================================

Una venta o reducción reservada no aumenta available capacity.

Antes de ejecución:

estimated_proceeds =
informativo únicamente.

Después de futura ejecución EOD:

realized_paper_proceeds =
official_close × executed_quantity

Solo entonces:

- Paper Cash aumenta;
- available capacity puede aumentar;
- una request PENDING_CAPITAL puede volver a previsualizarse;
- no se confirma automáticamente.

No encadenar automáticamente:

venta
→ compra

sin una nueva confirmación explícita o un flujo atómico aprobado
específicamente en un gate posterior.

==================================================
30. CAPITAL OBJETIVO Y CAPITAL EFECTIVO ASIGNADO
==================================================

El panel muestra:

Capital objetivo =
50.000,00

Capital efectivo asignado =
50.000,14

La diferencia de 0,14 puede proceder de:

- redondeo;
- valoración;
- representación agregada;
- reconstrucción.

No interpretarla como:

- Paper Cash;
- capacidad de compra;
- aportación;
- exceso disponible;
- autorización de leverage.

Gate 2F-A1 debe identificar y documentar la fuente de esos indicadores.

Entregar:

CAPITAL_TARGET_SOURCE =
<fuente>

EFFECTIVE_ALLOCATED_CAPITAL_SOURCE =
<fuente>

UNALLOCATED_CAPITAL_SOURCE =
<fuente>

HARD_CAP_CAPACITY_SOURCE =
LATEST_CANONICAL_PAPER_CASH_MINUS_ACTIVE_RESERVATIONS

No modificar estos indicadores durante este gate.

==================================================
31. MIGRACIÓN v30 EN LAB
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
H. estructuras económicas intactas;
I. alertas y catalizadores intactos;
J. posiciones y memberships intactos.

No aplicar a Producción.

==================================================
32. ÍNDICES Y CONSTRAINTS
==================================================

Definir:

A. Unicidad de idempotency keys.

B. Una sola cash reservation ACTIVE por allocation request.

C. Una sola quantity reservation ACTIVE por position action request.

D. amount > 0.

E. quantity > 0.

F. timestamps UTC.

G. effective_session nullable y compatible con NYSE.

H. currency coherente con cartera.

I. lifecycle events append-only.

J. foreign keys con RESTRICT o NO ACTION.

No usar ON DELETE CASCADE para borrar auditoría.

Añadir índices para consultar:

- requests por cartera y estado;
- actions por cartera y estado;
- actions por catalyst reference;
- reservations ACTIVE por cartera;
- quantity reservations ACTIVE por membership;
- events por request;
- idempotency keys.

==================================================
33. SERVICIOS INTERNOS
==================================================

Crear servicios desacoplados de HTTP y UI:

ASIGNACIÓN

- calculatePaperAvailableCapacity;
- previewPaperAllocation;
- confirmPaperAllocation;
- confirmPartialPaperAllocation;
- cancelPaperAllocation;
- getPaperAllocationLifecycle;
- getActivePaperCashReservations.

POSICIONES

- calculatePaperAvailableQuantity;
- reconcileCatalystAlertToPaperPosition;
- previewPaperPositionAction;
- acknowledgeHoldPosition;
- confirmPaperPositionReduction;
- confirmPaperFullExit;
- cancelPaperPositionAction;
- getPaperPositionActionLifecycle;
- getActivePaperQuantityReservations.

No crear todavía rutas públicas.

No integrar todavía con daily-close.

No integrar todavía con la UI productiva.

==================================================
34. AUDITORÍA
==================================================

Cada evento debe conservar:

- actor;
- timestamp UTC;
- old status;
- new status;
- request/action;
- cartera;
- posición;
- membership;
- ticker;
- catalyst reference;
- catalyst state;
- requested notional o quantity;
- capacidad observada;
- preview hash;
- confirmation reference;
- idempotency key;
- content hash.

No guardar:

- secretos;
- tokens;
- payloads innecesarios;
- información sensible no requerida.

Trazabilidad futura de compra:

tesis
→ allocation request
→ confirmación
→ cash reservation
→ futura transacción
→ futura composición
→ futuro snapshot

Trazabilidad futura de salida:

alerta de catalizador
→ decisión de Omar
→ position action request
→ confirmación
→ quantity reservation
→ futura venta/reducción
→ futura composición
→ futuro snapshot

==================================================
35. PRUEBAS
==================================================

Crear:

verify-paper-v30-migration.mjs
verify-paper-request-lifecycle.mjs
verify-paper-position-action-lifecycle.mjs
verify-paper-capacity-calculation.mjs
verify-paper-available-quantity.mjs
verify-paper-cash-reservation.mjs
verify-paper-quantity-reservation.mjs
verify-paper-full-confirmation.mjs
verify-paper-partial-confirmation.mjs
verify-paper-hold-decision.mjs
verify-paper-position-reduction-reservation.mjs
verify-paper-full-exit-reservation.mjs
verify-paper-reservation-cancellation.mjs
verify-paper-cash-reservation-concurrency.mjs
verify-paper-quantity-reservation-concurrency.mjs
verify-paper-reservation-idempotency.mjs
verify-paper-catalyst-alert-reconciliation.mjs
verify-paper-hard-cap-position-11.mjs
verify-paper-hard-cap-isolation.mjs

Casos obligatorios:

A. Migración v29 → v30.
B. Rollback vacío.
C. Requests históricos intactos.
D. Alertas y catalizadores intactos.
E. Preview de compra no escribe.
F. Preview de salida no escribe.
G. Cash suficiente reserva completo.
H. Cash insuficiente queda pending.
I. Asignación parcial exige confirmación.
J. Requested notional se preserva.
K. Mantener no crea reserva.
L. Reducción parcial reserva cantidad correcta.
M. Venta total reserva toda la cantidad disponible.
N. Dos salidas no reservan más cantidad de la existente.
O. Una venta reservada no crea Paper Cash.
P. Cancelación libera cash reservation.
Q. Cancelación libera quantity reservation.
R. Doble cancelación es idempotente.
S. Dos compras compiten por el mismo cash.
T. Dos ventas compiten por la misma cantidad.
U. Doble confirmación no duplica.
V. Mismo key con payload diferente falla.
W. Posición 11 queda PENDING_CAPITAL.
X. C1/C3 no se interpretan como carteras.
Y. Los siete tickers de alerta se reconcilian o quedan bloqueados
   individualmente.
Z. Cero composiciones creadas.
AA. Cero transacciones creadas.
AB. Cero memberships creados o cerrados.
AC. Cero snapshots creados.
AD. Política inactiva.
AE. Producción intacta.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- suites v27/v28/v29;
- Gate 2D-2;
- Gate 2E-A;
- financial core;
- repository integrity;
- catalizadores;
- tesis.

==================================================
36. AISLAMIENTO
==================================================

Después de las pruebas LAB verificar Producción:

- schema sigue v29;
- integrity_check;
- foreign_key_check;
- hard cap sigue sin effective_from;
- snapshots Papel idénticos;
- motor Papel idéntico;
- gráficas idénticas;
- holdings legacy idénticos;
- Book real idéntico;
- cash real idéntico;
- NAV real idéntico;
- tesis idénticas;
- catalizadores idénticos;
- alertas idénticas;
- Campo de Caza idéntico.

Resultados:

PRODUCTION_SCHEMA_UNCHANGED = PASS
PRODUCTION_DATA_UNCHANGED = PASS
PAPER_TRACK_UNCHANGED = PASS
CATALYST_ALERTS_UNCHANGED = PASS
THESES_UNCHANGED = PASS
REAL_BOOK_UNCHANGED = PASS
REAL_CASH_UNCHANGED = PASS
REAL_NAV_UNCHANGED = PASS
NO_POLICY_ACTIVATED = PASS

==================================================
37. CÓDIGO Y COMMIT
==================================================

Añadir únicamente:

- migración v30;
- rollback vacío;
- lifecycle append-only;
- cash reservations;
- quantity reservations;
- capacity calculator;
- available quantity calculator;
- allocation preview;
- position action preview;
- full allocation confirmation;
- partial allocation confirmation;
- hold acknowledgement;
- reduction reservation;
- full-exit reservation;
- cancellation;
- catalyst reconciliation;
- concurrency protection;
- pruebas;
- documentación técnica.

No añadir:

- rutas HTTP públicas;
- UI productiva;
- daily-close integration;
- catch-up integration;
- compras;
- ventas;
- composiciones prospectivas;
- memberships prospectivos;
- carteras nuevas;
- aportaciones;
- activación hard cap;
- scratchpad;
- backups;
- bases;
- secretos.

Ejecutar secret scan.

Crear un único commit local:

feat(lens): add atomic paper allocation and exit reservations

No crear tag.
No hacer push.

==================================================
38. ENTREGA
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
POSITION_ACTION_LIFECYCLE = PASS/FAIL

CAPACITY_CALCULATION = PASS/FAIL
AVAILABLE_QUANTITY_CALCULATION = PASS/FAIL

CASH_RESERVATION = PASS/FAIL
QUANTITY_RESERVATION = PASS/FAIL
CAPACITY_RACE_PROTECTION = PASS/FAIL
QUANTITY_RACE_PROTECTION = PASS/FAIL

FULL_ALLOCATION_CONFIRMATION = PASS/FAIL
PENDING_CAPITAL = PASS/FAIL
PARTIAL_ALLOCATION_CONFIRMATION = PASS/FAIL
PARTIAL_CONFIRMATION_REQUIRED = PASS/FAIL

HOLD_ACKNOWLEDGEMENT = PASS/FAIL
PARTIAL_EXIT_RESERVATION = PASS/FAIL
FULL_EXIT_RESERVATION = PASS/FAIL

CANCELLATION_RELEASES_CASH = PASS/FAIL
CANCELLATION_RELEASES_QUANTITY = PASS/FAIL

REQUEST_IDEMPOTENCY = PASS/FAIL
POSITION_ACTION_IDEMPOTENCY = PASS/FAIL
CASH_RESERVATION_IDEMPOTENCY = PASS/FAIL
QUANTITY_RESERVATION_IDEMPOTENCY = PASS/FAIL
CANCELLATION_IDEMPOTENCY = PASS/FAIL
IDEMPOTENCY_PAYLOAD_CONFLICT = PASS/FAIL

POSITION_11_PENDING = PASS/FAIL
AVAILABLE_CAPACITY_NEVER_NEGATIVE = PASS/FAIL
ACTIVE_CASH_RESERVATIONS_NEVER_EXCEED_CASH = PASS/FAIL
AVAILABLE_QUANTITY_NEVER_NEGATIVE = PASS/FAIL
ACTIVE_QUANTITY_RESERVATIONS_NEVER_EXCEED_POSITION = PASS/FAIL

CATALYST_ALERTS_RECONCILED = <número>/7
CATALYST_ALERTS_UNRESOLVED = <número>

CATALYST_CATEGORY_NOT_USED_AS_PORTFOLIO = PASS/FAIL
ALERT_DOES_NOT_AUTO_SELL = PASS/FAIL
RESERVED_EXIT_DOES_NOT_CREATE_CASH = PASS/FAIL

CAPITAL_TARGET_SOURCE = <fuente>
EFFECTIVE_ALLOCATED_CAPITAL_SOURCE = <fuente>
UNALLOCATED_CAPITAL_SOURCE = <fuente>

HARD_CAP_CAPACITY_SOURCE =
LATEST_CANONICAL_PAPER_CASH_MINUS_ACTIVE_RESERVATIONS

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
CATALYST_ALERTS_UNCHANGED = PASS/FAIL
THESES_UNCHANGED = PASS/FAIL
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