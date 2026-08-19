MIZAN · LA LENTE
GATE 2F-A2 · EJECUCIÓN ECONÓMICA PROSPECTIVA EOD
COMPRAS + REDUCCIONES + VENTAS · COMPOSICIONES · SNAPSHOTS
LAB ÚNICAMENTE · SIN HTTP · SIN UI · SIN ACTIVACIÓN

NATURALEZA

Esta ejecución autoriza exclusivamente en LAB:

1. consumir reservas ACTIVE de Paper Cash;
2. ejecutar compras confirmadas al cierre oficial;
3. consumir reservas ACTIVE de cantidad;
4. ejecutar reducciones parciales al cierre oficial;
5. ejecutar ventas completas al cierre oficial;
6. crear posiciones y memberships prospectivos cuando corresponda;
7. actualizar memberships después de reducciones o salidas;
8. crear composiciones EOD prospectivas;
9. crear transacciones económicas canónicas;
10. mantener continuidad de NAV y Paper Unit Value;
11. convertir ventas ejecutadas en Paper Cash;
12. actualizar snapshots del Track Papel;
13. actualizar estados de solicitudes y reservas;
14. implementar ejecución cronológica e idempotente;
15. modelar PENDING_MARKET_CLOSE y PENDING_PRICE;
16. probar recuperación tras interrupciones;
17. crear un único commit local si todo queda verde.

No autoriza:

- aplicar schema v30 en Producción;
- cablear todavía v30 en db.js productivo;
- activar MANUAL_NOTIONAL_HARD_CAP;
- fijar effective_from;
- modificar daily-close productivo;
- modificar startup catch-up productivo;
- crear endpoints HTTP públicos;
- modificar la UI;
- crear carteras nuevas;
- crear aportaciones externas;
- crear retiradas externas;
- crear reasignaciones entre carteras;
- ejecutar rebalanceos automáticos;
- utilizar ventas estimadas como capacidad;
- autoejecutar solicitudes PENDING_CAPITAL;
- modificar las 82 solicitudes históricas;
- modificar snapshots históricos existentes;
- modificar composiciones históricas;
- modificar transacciones históricas;
- modificar tesis;
- modificar catalizadores;
- modificar Campo de Caza;
- modificar Book, cash o NAV reales;
- desplegar;
- hacer push;
- crear tag;
- realizar operaciones Git remotas.

Todas las escrituras económicas deben realizarse en bases LAB aisladas.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = b651c7e
schema de Producción esperado = v29
schema LAB objetivo = v30
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Gate 2F-A1 confirmado:

V30_MIGRATION = PASS
REQUEST_LIFECYCLE = PASS
POSITION_ACTION_LIFECYCLE = PASS
CASH_RESERVATIONS = PASS
QUANTITY_RESERVATIONS = PASS
CAPACITY_RACE_PROTECTION = PASS
QUANTITY_RACE_PROTECTION = PASS
CATALYST_ALERT_RECONCILIATION = 7/7
TESTS = 101 PASS / 0 FAIL

Producción:

- permanece en v29;
- no contiene tablas v30;
- no contiene reservas;
- no contiene acciones prospectivas;
- hard cap sigue inactivo;
- effective_from sigue NULL.

Estado económico existente:

PAPER_PORTFOLIOS = 5
PAPER_GLOBAL_POSITIONS = 82
PAPER_MEMBERSHIP_EPISODES = 121
PAPER_TRACK_SNAPSHOTS = 54

PAPER_TRACK_METHOD =
PRICE_RETURN_TWR

CANONICAL_ECONOMIC_RECONSTRUCTION_VERSION =
1

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

PRICE_BASIS =
OFFICIAL_CLOSE_NOMINAL

==================================================
1. OBJETIVO FUNCIONAL
==================================================

Implementar en LAB estos flujos.

A. NUEVA POSICIÓN

RESERVED_FOR_EXECUTION
→ PENDING_MARKET_CLOSE
→ PENDING_PRICE
→ precio validado
→ compra
→ FULLY_ALLOCATED
→ reserva CONSUMED
→ composición EOD
→ snapshot
→ Track actualizado

B. ASIGNACIÓN PARCIAL

PARTIALLY_RESERVED_FOR_EXECUTION
→ PENDING_MARKET_CLOSE
→ PENDING_PRICE
→ compra parcial
→ PARTIALLY_ALLOCATED
→ reserva CONSUMED
→ composición EOD
→ snapshot

C. REDUCCIÓN

QUANTITY_RESERVED_FOR_EXECUTION
→ PENDING_MARKET_CLOSE
→ PENDING_PRICE
→ venta parcial
→ PARTIALLY_EXECUTED
→ reserva CONSUMED
→ membership permanece abierto
→ composición EOD
→ snapshot

D. VENTA COMPLETA

QUANTITY_RESERVED_FOR_EXECUTION
→ PENDING_MARKET_CLOSE
→ PENDING_PRICE
→ venta total
→ EXECUTED
→ reserva CONSUMED
→ membership cerrado
→ composición EOD
→ snapshot

E. PRECIO NO DISPONIBLE

reserva ACTIVE
→ PENDING_PRICE
→ cero ejecución
→ cero composición
→ cero transacción
→ cero snapshot nuevo
→ reintento posterior

==================================================
2. PRECONDICIONES
==================================================

Antes de modificar:

- confirmar HEAD b651c7e;
- confirmar Producción en v29;
- confirmar hard cap inactivo;
- confirmar effective_from NULL;
- confirmar una sola raíz Git;
- inventariar untracked;
- ejecutar integrity_check productivo;
- ejecutar foreign_key_check productivo;
- capturar hashes y contadores;
- crear copia LAB;
- aplicar schema v30 en LAB;
- ejecutar integrity_check LAB;
- ejecutar foreign_key_check LAB;
- ejecutar las 19 suites de Gate 2F-A1;
- confirmar 101 PASS / 0 FAIL.

Capturar baseline LAB:

- configuraciones;
- flujos;
- posiciones;
- memberships;
- composiciones;
- miembros;
- transacciones legacy;
- transacciones económicas;
- snapshots;
- requests;
- actions;
- reservas;
- manifiestos de precio;
- políticas.

No continuar si:

- Producción fue alterada;
- Gate 2F-A1 no está verde;
- existe más de una reconstrucción económica canónica;
- las reservas incumplen invariantes;
- el Track base no reconcilia.

Declarar:

PRODUCTION_READ_ONLY = PASS/FAIL
LAB_DATABASE_ISOLATED = PASS/FAIL

==================================================
3. ALCANCE DEL SCHEMA v30
==================================================

Auditar si el schema v30 de Gate 2F-A1 y las tablas v29 existentes soportan:

- execution batch;
- execution attempt;
- effective session;
- executed quantity;
- executed price;
- executed notional;
- transaction reference;
- composition reference;
- snapshot reference;
- failure code;
- retryable status;
- membership close reason;
- source reservation;
- source catalyst alert;
- content hash;
- idempotency key.

Si ya existe soporte suficiente:

- no ampliar schema.

Si falta soporte:

- ampliar el diseño v30 únicamente de forma aditiva;
- mantener intactas las siete tablas creadas en 2F-A1;
- no reconstruir tablas históricas;
- no alterar las 82 solicitudes históricas;
- probar de nuevo migración e idempotencia.

Como v30 todavía no se ha aplicado en Producción, puede completarse antes de
su despliegue siempre que:

- la ampliación sea aditiva;
- las pruebas A1 sigan verdes;
- no cambie la semántica de campos existentes;
- la migración siga siendo v29 → v30 transaccional;
- se documente el cambio.

No crear v31 salvo que exista evidencia de que v30 fue aplicada fuera de
copias LAB efímeras.

Puede añadirse:

paper_prospective_execution

Campos mínimos:

- execution_id;
- execution_type;
- allocation_request_id nullable;
- position_action_request_id nullable;
- cash_reservation_id nullable;
- quantity_reservation_id nullable;
- paper_portfolio_id;
- global_position_id nullable;
- membership_episode_id nullable;
- ticker;
- effective_session;
- status;
- requested_quantity nullable;
- executed_quantity nullable;
- executed_price nullable;
- executed_notional nullable;
- cash_delta nullable;
- composition_version_id nullable;
- transaction_id nullable;
- snapshot_id nullable;
- attempt_count;
- last_attempt_at nullable;
- failure_code nullable;
- failure_reason nullable;
- idempotency_key;
- content_hash.

Estados:

- PENDING_MARKET_CLOSE;
- PENDING_PRICE;
- READY;
- EXECUTING;
- EXECUTED;
- FAILED_RETRYABLE;
- FAILED_FINAL;
- CANCELLED.

==================================================
4. FUENTES ECONÓMICAS CANÓNICAS
==================================================

Para ejecutar una reserva usar exclusivamente:

- último snapshot canónico anterior a la sesión;
- composición económica EOD vigente;
- cantidades económicas canónicas;
- Paper Cash canónico;
- reconstruction version canónica;
- reserva ACTIVE;
- request/action lifecycle;
- cierre oficial validado;
- calendario NYSE;
- manifiesto versionado.

No utilizar:

- configured_capital como NAV;
- allocated_notional legacy;
- quantity legacy;
- precio de entrada;
- precio live;
- previous close;
- adjusted total-return;
- Capital objetivo agregado;
- Capital efectivo asignado agregado;
- proceeds estimados;
- cash real;
- NAV real.

==================================================
5. RESOLUCIÓN DE LA SESIÓN EFECTIVA
==================================================

Usar America/New_York y el calendario NYSE.

A. Confirmación antes de apertura:

effective_session =
sesión bursátil actual.

B. Confirmación durante sesión abierta:

effective_session =
sesión bursátil actual.

C. Confirmación después del cierre:

effective_session =
siguiente sesión bursátil.

D. Fin de semana/festivo:

effective_session =
siguiente sesión bursátil.

E. Early close:

- confirmación antes del cierre específico → sesión actual;
- confirmación después → siguiente sesión.

No ejecutar una operación con un cierre anterior a:

confirmed_at_utc

No codificar una fecha fija.

Las pruebas deben utilizar relojes y calendarios inyectables.

==================================================
6. TRANSICIÓN A PENDING_MARKET_CLOSE
==================================================

Después de confirmar y reservar:

RESERVED_FOR_EXECUTION

o:

PARTIALLY_RESERVED_FOR_EXECUTION

debe pasar a:

PENDING_MARKET_CLOSE

cuando exista una effective_session determinada.

Las acciones de reducción/venta:

QUANTITY_RESERVED_FOR_EXECUTION
→ PENDING_MARKET_CLOSE

Esta transición:

- no consume reserva;
- no crea posición;
- no crea transacción;
- no crea composición;
- no crea snapshot;
- no crea Paper Cash;
- no cambia NAV.

Debe ser append-only e idempotente.

==================================================
7. PRECIO Y MANIFIESTO
==================================================

Después del cierre de la effective_session:

1. comprobar que la sesión está cerrada;
2. buscar evidencia sellada;
3. si no existe, obtenerla mediante el proveedor canónico;
4. validar ticker, sesión, moneda y precio;
5. extender el manifiesto de forma versionada;
6. calcular nuevo hash;
7. conservar inmutable la evidencia anterior;
8. marcar ejecución READY.

Si falta el precio:

execution status =
PENDING_PRICE

request/action status =
PENDING_PRICE

La reserva permanece ACTIVE.

No usar fallback.

No crear transacción ni snapshot.

El reintento debe poder ejecutarse posteriormente.

==================================================
8. ORDEN ECONÓMICO DE UNA SESIÓN
==================================================

Por cartera y sesión:

1. cargar snapshot anterior;
2. cargar cantidades anteriores;
3. cargar Paper Cash anterior;
4. marcar posiciones existentes al cierre actual;
5. calcular NAV_PRE_FLOW;
6. aplicar flujos externos efectivos, si existieran;
7. calcular NAV_PRE_TRADE;
8. cargar reservas ejecutables;
9. validar nuevamente cash y cantidades;
10. ejecutar ventas/reducciones;
11. ejecutar compras;
12. crear composición EOD final;
13. calcular Paper Cash final;
14. calcular market value final;
15. calcular NAV_POST_TRADE;
16. comprobar continuidad;
17. crear snapshot;
18. consumir reservas;
19. actualizar lifecycle;
20. COMMIT.

En Gate 2F-A2 no existirán nuevos flujos externos, salvo fixtures puramente
LAB para probar el orden económico.

==================================================
9. ORDEN ENTRE VENTAS Y COMPRAS
==================================================

Dentro de una misma cartera y sesión:

- procesar primero reducciones y ventas confirmadas;
- después procesar compras confirmadas.

Pero los proceeds de una venta solo pueden financiar una compra de la misma
sesión si existe un flujo atómico explícitamente confirmado que vincule ambas
operaciones.

Gate 2F-A2 no autoriza ese encadenamiento automático.

Por defecto:

- una venta ejecutada aumenta Paper Cash al final de la sesión;
- una compra que estaba PENDING_CAPITAL permanece pendiente;
- no se crea automáticamente una reserva de cash;
- no se autoejecuta la nueva tesis.

Después de la venta, Omar deberá realizar:

- nuevo preview;
- nueva confirmación;
- nueva reserva.

Declarar:

SALE_DOES_NOT_AUTO_TRIGGER_PENDING_BUY = PASS/FAIL

==================================================
10. EJECUCIÓN DE NUEVA POSICIÓN
==================================================

Para request_type:

NEW_POSITION

y cash reservation ACTIVE:

1. validar request;
2. validar effective session;
3. validar precio;
4. validar Paper Cash;
5. validar que el ticker/tesis no tenga ya una incorporación equivalente;
6. crear paper_global_position;
7. crear membership económico;
8. calcular quantity;
9. crear transacción PURCHASE;
10. crear composición EOD;
11. consumir cash reservation;
12. actualizar request;
13. crear snapshot.

Fórmula:

executed_notional =
reservation.amount

executed_quantity =
executed_notional / official_close

cash_delta =
-executed_notional

No redondear quantity de forma que se exceda la reserva.

Si la precisión de quantity provoca residuo:

- preservar Paper Cash residual;
- no aumentar notional;
- documentar rounding residual.

La nueva posición comienza exposición al cierre de esa sesión.

No recibe retorno anterior a la compra.

Su primera variación de precio ocurre en una sesión posterior.

==================================================
11. EJECUCIÓN DE AMPLIACIÓN
==================================================

Para request_type:

INCREASE_POSITION

si se implementa en este gate:

- utilizar membership económico abierto;
- conservar requested_notional original;
- calcular quantity adicional;
- crear PURCHASE;
- mantener membership abierto;
- actualizar composición;
- consumir reserva;
- actualizar snapshot.

No recalcular ni normalizar las demás posiciones.

No modificar su cantidad.

Si INCREASE_POSITION no puede implementarse de forma completa y segura:

- dejarlo explícitamente fuera de alcance;
- bloquearlo con NOT_IMPLEMENTED;
- no simularlo como NEW_POSITION.

NEW_POSITION es obligatorio.

INCREASE_POSITION es opcional en Gate 2F-A2.

==================================================
12. COMPOSICIÓN DESPUÉS DE UNA COMPRA
==================================================

La composición EOD final debe contener:

- todas las posiciones económicas anteriores;
- sus cantidades anteriores, salvo otras operaciones confirmadas;
- la nueva posición;
- su cantidad ejecutada;
- Paper Cash final;
- pesos observados.

No aplicar:

- equal-weight;
- capital/N;
- normalización;
- rebalanceo automático;
- dilución de cantidades existentes.

Los pesos son consecuencia de:

market_value_i / Paper_NAV

No son objetivos automáticos.

==================================================
13. EJECUCIÓN DE REDUCCIÓN PARCIAL
==================================================

Para:

PARTIAL_EXIT

y quantity reservation ACTIVE:

1. validar membership abierto;
2. cargar canonical quantity;
3. validar quantity reservation;
4. validar precio;
5. comprobar que reserved quantity sigue disponible;
6. crear SALE por la cantidad reservada;
7. reducir quantity;
8. mantener membership abierto;
9. aumentar Paper Cash;
10. crear composición EOD;
11. consumir quantity reservation;
12. actualizar action lifecycle;
13. crear snapshot.

Fórmulas:

executed_quantity =
reserved_quantity

executed_notional =
executed_quantity × official_close

cash_delta =
+executed_notional

remaining_quantity =
quantity_before - executed_quantity

Debe cumplirse:

remaining_quantity > 0

Si el redondeo deja una cantidad económicamente cero:

- no convertir silenciosamente una reducción en salida total;
- bloquear y exigir FULL_EXIT explícita;
- o aplicar una regla de dust previamente documentada y confirmada.

Preferencia:

no convertir automáticamente.

==================================================
14. EJECUCIÓN DE VENTA TOTAL
==================================================

Para:

FULL_EXIT

y quantity reservation ACTIVE:

1. validar membership abierto;
2. cargar canonical quantity;
3. validar que la reserva cubre toda available quantity;
4. validar que no existen cantidades sin reservar incompatibles;
5. validar precio;
6. crear SALE;
7. establecer cantidad final cero;
8. cerrar membership;
9. mantener global position histórica;
10. aumentar Paper Cash;
11. crear composición EOD sin esa posición;
12. consumir reservation;
13. actualizar lifecycle a EXECUTED;
14. crear snapshot.

Campos de cierre:

- exited_at_utc;
- effective_exit_session;
- exit_price;
- exit_reason;
- catalyst reference;
- position action request;
- transaction reference;
- content hash.

exit_reason puede ser:

CATALYST_NO_LONGER_ACTIVE
CATALYST_REVERTED
CATALYST_COMPLETED
OWNER_DECISION
OTHER_EXPLICIT_REASON

No modificar el estado del catalizador como efecto de la venta.

==================================================
15. OPCIÓN MANTENER
==================================================

Las decisiones:

HOLD_ACKNOWLEDGED

no entran en el motor EOD.

No crean:

- ejecución;
- transacción;
- composición;
- snapshot;
- cash;
- cambio de quantity.

El motor debe ignorarlas económicamente.

La alerta puede reaparecer posteriormente si se cumplen las reglas de revisión.

==================================================
16. NAV Y UNIT VALUE EN COMPRAS
==================================================

Para una compra interna:

NAV_PRE_TRADE =
Paper Cash antes
+ market value antes.

Después:

Paper Cash disminuye.
Market Value aumenta.

Debe cumplirse:

NAV_POST_TRADE =
NAV_PRE_TRADE

Paper Units después =
Paper Units antes

Paper Unit Value después =
Paper Unit Value antes

Internal trade return =
0

La compra no genera rendimiento.

==================================================
17. NAV Y UNIT VALUE EN VENTAS
==================================================

Para reducción o venta:

antes:

market value incluye la cantidad vendida al cierre.

después:

- market value disminuye;
- Paper Cash aumenta por el mismo importe.

Debe cumplirse:

NAV_POST_TRADE =
NAV_PRE_TRADE

Paper Units después =
Paper Units antes

Paper Unit Value después =
Paper Unit Value antes

Internal trade return =
0

La venta no crea una ganancia o pérdida adicional.

El movimiento de mercado hasta el cierre ya forma parte de NAV_PRE_TRADE.

==================================================
18. PAPER CASH DESPUÉS DE UNA SALIDA
==================================================

Los proceeds ejecutados quedan como:

Paper Cash

según:

EXIT_POLICY =
KEEP_AS_PAPER_CASH

No crear:

- retirada externa;
- compra automática;
- aportación;
- unidades;
- transferencia real;
- operación SPY.

El benchmark SPY no cambia por ventas internas.

El Paper Cash nuevo será visible en:

- snapshot;
- endpoint;
- LensPaperTrackView;
- futuros cálculos de capacidad.

Pero no reserva automáticamente ese cash para solicitudes pendientes.

==================================================
19. SNAPSHOT DE LA SESIÓN
==================================================

Crear un único snapshot por:

- portfolio;
- methodology;
- session;
- canonical economic version.

El snapshot debe reflejar:

- cantidades finales;
- Paper Cash final;
- market value final;
- Paper NAV;
- Paper Units;
- Paper Unit Value;
- retorno diario;
- retorno acumulado;
- drawdown;
- SPY;
- cobertura;
- confianza;
- posiciones activas.

No crear un snapshot por cada operación.

Todas las operaciones de la misma cartera y sesión deben consolidarse en una
única composición EOD y un único snapshot.

Si ya existe snapshot para esa sesión en LAB:

- no modificarlo silenciosamente;
- utilizar fixture que termine antes de la sesión;
- o crear una nueva versión económica prospectiva explícita;
- documentar la estrategia.

Para pruebas, preferir sesiones futuras simuladas posteriores al último
snapshot fixture.

==================================================
20. CONFIANZA HISTÓRICA Y PROSPECTIVA
==================================================

Una posición prospectiva creada después de una confirmación completa tiene:

historical_confidence =
FULL_HISTORICAL_CONFIDENCE

si:

- tesis identificada;
- requested_notional conocido;
- reserva conocida;
- precio validado;
- ejecución conocida;
- membership conocido.

No cambiar retrospectivamente la confianza de las 36 posiciones históricas.

Un snapshot posterior puede seguir siendo:

PARTIAL_HISTORICAL_CONFIDENCE

si mantiene posiciones históricas inferidas activas.

La nueva posición prospectiva no elimina esa advertencia.

==================================================
21. ALERTAS ACTUALES
==================================================

En LAB probar al menos:

CATALYST_NO_LONGER_ACTIVE:

- ONB;
- ATRC;
- ISRG;
- HCSG;
- WSBC;
- WTFC.

CATALYST_REVERTED:

- PEGA.

No ejecutar en Producción.

Crear fixtures LAB para demostrar:

- HOLD sobre una alerta;
- PARTIAL_EXIT sobre una alerta;
- FULL_EXIT sobre una alerta;
- cancelación antes del cierre;
- precio faltante;
- doble confirmación;
- doble venta;
- continuidad NAV.

No es obligatorio ejecutar las siete salidas.

Sí es obligatorio que las siete permanezcan reconciliadas y elegibles según
su estado económico real.

==================================================
22. RESERVAS Y CONSUMO
==================================================

Una reserva ACTIVE puede pasar a:

CONSUMED

solo dentro de la misma transacción económica que crea:

- transacción;
- composición;
- snapshot;
- lifecycle final.

No marcar CONSUMED antes de completar la operación.

Si falla:

- rollback;
- reserva sigue ACTIVE;
- request/action sigue pendiente;
- no queda ejecución parcial.

Una reserva RELEASED, EXPIRED o INVALIDATED no puede ejecutarse.

==================================================
23. VALIDACIÓN TRANSACCIONAL PREVIA
==================================================

Justo antes de ejecutar, dentro de la transacción:

PARA COMPRA

- cash reservation ACTIVE;
- amount válido;
- Paper Cash suficiente;
- membership compatible;
- request no cancelada;
- precio validado;
- sesión correcta.

PARA VENTA

- quantity reservation ACTIVE;
- quantity válida;
- membership abierto;
- canonical quantity suficiente;
- action no cancelada;
- precio validado;
- sesión correcta.

Si una validación falla:

- no ejecutar;
- clasificar retryable/final;
- conservar auditoría;
- no corromper reservas.

==================================================
24. IDEMPOTENCIA
==================================================

Una ejecución repetida debe producir:

ALREADY_EXECUTED

y no crear:

- segunda posición;
- segundo membership;
- segunda transacción;
- segunda composición;
- segundo snapshot;
- segundo consumo de reserva;
- segundo cierre.

Claves de idempotencia por:

- request/action;
- portfolio;
- effective session;
- execution type;
- economic version.

Probar interrupciones en:

A. antes de BEGIN;
B. después de BEGIN;
C. después de crear transacción;
D. después de crear composición;
E. antes de snapshot;
F. antes de COMMIT;
G. después de COMMIT.

Solo dos resultados son válidos:

- operación completa;
- cero operación.

==================================================
25. CONCURRENCIA DE EJECUCIÓN
==================================================

Probar:

- dos workers procesan la misma reserva;
- daily-close y catch-up procesan la misma reserva;
- cancelación compite con ejecución;
- dos operaciones de la misma cartera y sesión;
- dos operaciones sobre la misma posición.

Utilizar:

BEGIN IMMEDIATE

o mecanismo equivalente.

Una sola ejecución puede ganar.

La otra debe devolver:

ALREADY_EXECUTED
EXECUTION_IN_PROGRESS
STATE_CONFLICT

según corresponda.

Nunca duplicar el efecto económico.

==================================================
26. CANCELACIÓN FRENTE A EJECUCIÓN
==================================================

Si CANCELLED confirma antes de que la ejecución adquiera el bloqueo:

- liberar reserva;
- no ejecutar.

Si la ejecución adquiere el bloqueo y completa primero:

- la cancelación posterior devuelve ALREADY_EXECUTED;
- no revierte historia;
- no borra transacción.

No permitir estado:

CANCELLED + EXECUTED

para la misma solicitud canónica.

==================================================
27. PENDING_PRICE
==================================================

Si la sesión está cerrada pero falta precio:

- request/action → PENDING_PRICE;
- execution → PENDING_PRICE;
- reserva sigue ACTIVE;
- no crear transacción;
- no crear composición;
- no crear snapshot modificado.

Cuando aparezca el precio:

- validar;
- reintentar;
- ejecutar una sola vez.

No utilizar precio de una sesión posterior para ejecutar retroactivamente.

Debe utilizarse el cierre exacto de la effective_session.

==================================================
28. EJECUCIÓN CRONOLÓGICA
==================================================

Crear servicio interno:

processPendingPaperExecutions

Debe:

- recibir portfolio y hasta una sesión cerrada;
- localizar reservas ejecutables;
- ordenar por effective session;
- ordenar por confirmed_at;
- utilizar request/action ID como desempate;
- procesar cronológicamente;
- detener una cartera ante un hueco obligatorio;
- permitir que otras carteras continúen;
- ser idempotente.

No cablear todavía este servicio al scheduler productivo.

Gate 2F-B hará la integración con daily-close y catch-up.

==================================================
29. SERVICIOS INTERNOS
==================================================

Crear o completar:

- schedulePaperAllocationExecution;
- schedulePaperPositionActionExecution;
- getPendingPaperExecutions;
- validatePaperExecutionReadiness;
- executePaperNewPosition;
- executePaperPositionIncrease, si se implementa;
- executePaperPartialExit;
- executePaperFullExit;
- buildProspectivePaperComposition;
- persistProspectivePaperTransaction;
- persistProspectivePaperSnapshot;
- processPendingPaperExecutions;
- retryPendingPriceExecution;
- getPaperExecutionLifecycle.

Todos deben probarse directamente sin HTTP ni UI.

==================================================
30. PRUEBAS
==================================================

Crear:

verify-paper-prospective-execution-schema.mjs
verify-paper-execution-session-resolution.mjs
verify-paper-execution-pending-market-close.mjs
verify-paper-execution-pending-price.mjs
verify-paper-execution-new-position.mjs
verify-paper-execution-partial-allocation.mjs
verify-paper-execution-partial-exit.mjs
verify-paper-execution-full-exit.mjs
verify-paper-execution-membership.mjs
verify-paper-execution-composition.mjs
verify-paper-execution-cash.mjs
verify-paper-execution-nav-neutrality.mjs
verify-paper-execution-unit-neutrality.mjs
verify-paper-execution-snapshot.mjs
verify-paper-execution-confidence.mjs
verify-paper-execution-idempotency.mjs
verify-paper-execution-concurrency.mjs
verify-paper-execution-cancellation-race.mjs
verify-paper-execution-chronology.mjs
verify-paper-execution-isolation.mjs

Casos obligatorios:

A. Compra completa.
B. Compra parcial.
C. Paper Cash insuficiente bloquea ejecución.
D. Nueva posición crea global position.
E. Nueva posición crea membership.
F. Compra no modifica posiciones existentes.
G. Compra no rebalancea.
H. Compra NAV-neutral.
I. Compra Unit Value-neutral.
J. Compra sin precio queda pending.
K. Reducción parcial mantiene membership.
L. Reducción aumenta Paper Cash.
M. Venta total cierra membership.
N. Venta total conserva global position histórica.
O. Venta NAV-neutral.
P. Venta Unit Value-neutral.
Q. SPY no cambia por compra/venta interna.
R. Hold no ejecuta nada.
S. Reserva se consume solo con operación completa.
T. Cancelación libera y bloquea ejecución.
U. Doble ejecución no duplica.
V. Dos workers no duplican.
W. Same-session operations crean una composición.
X. Same-session operations crean un snapshot.
Y. Venta no autoejecuta compra pendiente.
Z. Proceeds no existen antes de ejecución.
AA. Confianza histórica previa intacta.
AB. Siete alertas siguen reconciliadas.
AC. Producción permanece v29.
AD. Política permanece inactiva.
AE. Cero escrituras reales.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- 19 suites Gate 2F-A1;
- suites v27/v28/v29;
- Gate 2D-1;
- Gate 2D-2;
- Gate 2E-A;
- financial core;
- repository integrity.

==================================================
31. AISLAMIENTO DE PRODUCCIÓN
==================================================

Después de las pruebas:

- Producción sigue en v29;
- integrity_check;
- foreign_key_check;
- hard cap sin effective_from;
- cero tablas nuevas productivas;
- cero reservas productivas;
- cero executions productivas;
- snapshots productivos idénticos;
- Track Papel productivo idéntico;
- gráficas idénticas;
- alertas idénticas;
- tesis idénticas;
- catalizadores idénticos;
- Book/cash/NAV reales idénticos.

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
32. CÓDIGO Y COMMIT
==================================================

Añadir exclusivamente:

- soporte de ejecución prospectiva LAB;
- estado de ejecuciones;
- resolución de sesión;
- consumo de reservas;
- compras;
- reducciones;
- ventas;
- composiciones EOD prospectivas;
- transacciones prospectivas;
- snapshots LAB;
- continuidad NAV/Unit Value;
- idempotencia;
- concurrencia;
- pruebas;
- documentación técnica.

No añadir:

- endpoints públicos;
- UI;
- integración scheduler productiva;
- creación de carteras nuevas;
- aportaciones productivas;
- reasignaciones;
- activación de política;
- cambios productivos de schema;
- scratchpad;
- backups;
- bases;
- secretos.

Ejecutar secret scan.

Crear un único commit local:

feat(lens): add NAV-neutral paper EOD execution engine

No crear tag.
No hacer push.

==================================================
33. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = b651c7e
CANONICAL_HEAD_AFTER = <hash>

PRODUCTION_SCHEMA_BEFORE = v29
PRODUCTION_SCHEMA_AFTER = v29

LAB_SCHEMA_BEFORE = v29
LAB_SCHEMA_AFTER = v30

V30_EXECUTION_SUPPORT =
EXISTING /
ADDITIVELY_EXTENDED

EXECUTION_TABLES_ADDED = <lista>
EXECUTION_INDEXES_ADDED = <número>

NEW_POSITION_EXECUTION = PASS/FAIL
PARTIAL_ALLOCATION_EXECUTION = PASS/FAIL
PARTIAL_EXIT_EXECUTION = PASS/FAIL
FULL_EXIT_EXECUTION = PASS/FAIL

PENDING_MARKET_CLOSE = PASS/FAIL
PENDING_PRICE_FAIL_CLOSED = PASS/FAIL
EXACT_EFFECTIVE_SESSION_PRICE = PASS/FAIL

PURCHASE_NAV_NEUTRAL = PASS/FAIL
PURCHASE_UNIT_NEUTRAL = PASS/FAIL
PARTIAL_EXIT_NAV_NEUTRAL = PASS/FAIL
PARTIAL_EXIT_UNIT_NEUTRAL = PASS/FAIL
FULL_EXIT_NAV_NEUTRAL = PASS/FAIL
FULL_EXIT_UNIT_NEUTRAL = PASS/FAIL

NEW_POSITION_MEMBERSHIP_CREATED = PASS/FAIL
PARTIAL_EXIT_MEMBERSHIP_REMAINS_OPEN = PASS/FAIL
FULL_EXIT_MEMBERSHIP_CLOSED = PASS/FAIL
GLOBAL_POSITION_HISTORY_PRESERVED = PASS/FAIL

PROSPECTIVE_COMPOSITION_CREATED = PASS/FAIL
PROSPECTIVE_TRANSACTION_CREATED = PASS/FAIL
PROSPECTIVE_SNAPSHOT_CREATED = PASS/FAIL

CASH_RESERVATION_CONSUMED_ATOMICALLY = PASS/FAIL
QUANTITY_RESERVATION_CONSUMED_ATOMICALLY = PASS/FAIL
FAILED_EXECUTION_PRESERVES_RESERVATION = PASS/FAIL

SALE_PROCEEDS_TO_PAPER_CASH = PASS/FAIL
SALE_DOES_NOT_AUTO_TRIGGER_PENDING_BUY = PASS/FAIL
SPY_IGNORES_INTERNAL_TRADES = PASS/FAIL

EXECUTION_IDEMPOTENCY = PASS/FAIL
EXECUTION_CONCURRENCY = PASS/FAIL
CANCELLATION_EXECUTION_RACE = PASS/FAIL
CHRONOLOGICAL_PROCESSING = PASS/FAIL

CATALYST_ALERTS_RECONCILED = <número>/7
HOLD_CREATES_NO_EXECUTION = PASS/FAIL

TESTS_PASS = <número>
TESTS_FAIL = 0

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

EXACT_BLOCKERS_BEFORE_GATE_2F_B:
1. <bloqueo>
2. <bloqueo>
3. <bloqueo>

Detenerse después de Gate 2F-A2.