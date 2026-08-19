MIZAN · LA LENTE
GATE 2C-2 · BACKFILL HISTÓRICO ESTRUCTURAL POINT-IN-TIME
POSICIONES · MEMBERSHIPS · COMPOSICIONES · FLUJOS · TRANSACCIONES
SIN SERIE DIARIA · SIN UI · SIN ACTIVACIÓN PROSPECTIVA

NATURALEZA

Esta ejecución autoriza:

1. corregir el soporte de sesiones de cierre anticipado;
2. validar los artefactos de precios de Gate 2C-1;
3. reconstruir el historial AS_OPERATED;
4. generar un preview final de filas;
5. realizar backup validado de Producción;
6. crear el backfill estructural v27;
7. poblar configuraciones históricas por cartera;
8. crear posiciones globales;
9. crear memberships históricos;
10. crear solicitudes conocidas y desconocidas;
11. crear versiones de composición;
12. crear miembros de composición;
13. crear flujos externos iniciales;
14. crear transacciones internas legacy;
15. conservar estados de confianza y certificación;
16. crear un único commit local.

No autoriza:

- crear paper_track_snapshot;
- reconstruir la serie diaria;
- modificar daily-close;
- modificar startup catch-up;
- crear endpoints funcionales del Track;
- modificar la UI;
- modificar el Track de Crecimiento;
- activar MANUAL_NOTIONAL_HARD_CAP;
- establecer effective_from para la política prospectiva;
- aplicar la política hard cap a solicitudes actuales;
- modificar holdings legacy;
- modificar snapshots legacy;
- modificar cat_composicion_log;
- modificar tesis;
- modificar catalizadores;
- modificar Campo de Caza;
- crear operaciones reales;
- conectar con Wio;
- tag;
- push;
- operaciones Git remotas.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = 4c97712
SCHEMA esperado = v27
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Gate 2C-1 confirmado:

COVERAGE_END_SESSION = 2026-07-24
PRICE_PROVIDER = FMP historical-price-eod
PRICE_BASIS = OFFICIAL_CLOSE_NOMINAL

RAW_PRICE_REQUIREMENTS = 948
UNIQUE_PRICE_REQUIREMENTS = 863
VALIDATED_PRICE_POINTS = 863
PRICE_COVERAGE = 100 %

ANCHORS_VALIDATED = 82/82
REASSIGNMENTS_PRICE_VALIDATED = 39/39
DAILY_PRICE_POINTS_VALIDATED = 812/812
SPY_SESSIONS_VALIDATED = 15/15

Inventario histórico:

GLOBAL_PAPER_POSITIONS = 82

PORTFOLIO_MEMBERSHIP_EPISODES = 121
CURRENT_OPEN_MEMBERSHIP_EPISODES = 82
CLOSED_MEMBERSHIP_EPISODES = 39

INTER_PORTFOLIO_REASSIGNMENTS = 39
ECONOMICALLY_CLOSED_GLOBAL_POSITIONS = 0

REQUESTED_NOTIONAL_KNOWN = 46
REQUESTED_NOTIONAL_UNKNOWN_PRE_LOG = 26
REQUESTED_NOTIONAL_UNKNOWN_UNLINKED = 10

Metodología:

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

RETURN_METHOD =
PRICE_RETURN_TWR

EXIT_POLICY =
KEEP_AS_PAPER_CASH

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

COMMISSION_POLICY =
NOT_MODELED

BENCHMARK =
SPY

BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

==================================================
1. PRECONDICIONES
==================================================

Antes de cualquier escritura:

- confirmar HEAD 4c97712;
- confirmar env-info;
- confirmar una única raíz Git;
- inventariar archivos untracked;
- confirmar schema v27;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- confirmar una sola instancia;
- confirmar health;
- verificar definiciones metodológicas;
- verificar que MANUAL_NOTIONAL_HARD_CAP sigue sin effective_from;
- verificar que las nueve tablas operativas siguen vacías;
- validar hashes de los artefactos de Gate 2C-1;
- comprobar que staging no fue alterado después de la validación.

Enumerar expresamente:

1. paper_portfolio_track_config;
2. paper_global_position;
3. paper_portfolio_membership_episode;
4. paper_allocation_request;
5. paper_composition_version;
6. paper_composition_member;
7. paper_portfolio_flow;
8. paper_portfolio_transaction;
9. paper_track_snapshot.

Antes del backfill, las nueve deben contener cero filas.

Capturar para comparación posterior:

- movimientos;
- cash_events;
- holdings;
- snapshots;
- valuations;
- carteras_papel;
- cat_composicion_log;
- tesis;
- catalizadores;
- Track de Crecimiento;
- cash real;
- NAV real.

Capturar además:

- payload de /track/crecimiento;
- métricas;
- ventanas;
- benchmark;
- drawdown;
- última sesión;
- hash de valuations de Crecimiento.

No continuar si:

- staging no coincide con sus hashes;
- falta un precio obligatorio;
- alguna tabla operativa contiene filas;
- la política prospectiva está activa;
- integrity_check o foreign_key_check falla.

==================================================
2. CIERRES ANTICIPADOS
==================================================

Antes de resolver definitivamente eventos temporales, extender el calendario
canónico para soportar cierres anticipados NYSE.

No utilizar siempre 16:00 America/New_York.

El calendario debe resolver por sesión:

- regular_open;
- regular_close;
- early_close;
- holiday;
- non-trading day.

Para una sesión con cierre anticipado:

- una incorporación anterior al cierre anticipado es REGULAR_OPEN;
- una incorporación posterior es AFTER_CLOSE;
- el cierre oficial puede ser anchor desde que esté disponible;
- no esperar ficticiamente hasta las 16:00.

Probar como mínimo:

- Black Friday;
- víspera de Independence Day cuando corresponda;
- otra media sesión del calendario soportado;
- sesión regular;
- festivo;
- fin de semana.

Verificar que ninguna entrada o reasignación de la ventana histórica cambia
por este soporte.

Entregar:

EARLY_CLOSE_EVENTS_IN_HISTORY = <número>
ANCHORS_CHANGED_BY_EARLY_CLOSE = <número>
REASSIGNMENTS_CHANGED_BY_EARLY_CLOSE = <número>

El resultado esperado según Gate 2C-1 es cero, pero debe demostrarse.

==================================================
3. VALIDACIÓN DE LOS 121 MEMBERSHIPS
==================================================

La cobertura debe evaluarse sobre los 121 membership episodes, no únicamente
sobre las 82 posiciones actualmente abiertas.

Para cada membership:

- global_position_id;
- paper_portfolio_id;
- entered_at;
- entered_session;
- exited_at, cuando exista;
- exited_session, cuando exista;
- required first session;
- required last session;
- price points required;
- price points validated;
- coverage status.

Para los 39 cerrados por reasignación:

- validar todo el intervalo en la cartera de origen;
- validar el precio de salida de composición;
- validar el mismo cierre para la entrada en destino;
- mantener reassignment_group_id.

Declarar:

MEMBERSHIP_EPISODES_TOTAL = 121
MEMBERSHIP_EPISODES_PRICE_COMPLETE = <número>
OPEN_MEMBERSHIPS_PRICE_COMPLETE = <número>
CLOSED_MEMBERSHIPS_PRICE_COMPLETE = <número>

No iniciar backfill salvo:

MEMBERSHIP_EPISODES_PRICE_COMPLETE = 121

==================================================
4. ACCIONES CORPORATIVAS
==================================================

Clasificar exactamente cada señal detectada.

Para IMMR y cualquier otro caso entregar:

- ticker;
- effective session;
- event type;
- dividend;
- split ratio;
- symbol change;
- merger/acquisition;
- other;
- source;
- effect on close;
- effect on quantity;
- effect on PRICE_RETURN_TWR;
- required handling;
- blocking status.

Si es DIVIDEND:

- no añadir ingreso;
- no usar adjusted total-return;
- mantener cierre nominal;
- registrar warning INCOME_NOT_INCLUDED;
- aceptar que el precio ex-dividend puede reducir PRICE_RETURN_TWR.

Si es SPLIT:

- ajustar cantidad;
- preservar valor económico;
- no generar retorno artificial;
- registrar SPLIT_ADJUSTMENT.

Si el tipo no puede demostrarse:

- bloquear el episodio afectado;
- no clasificarlo como neutral por defecto.

Entregar:

CORPORATE_ACTIONS_CLASSIFIED = PASS/FAIL
UNRESOLVED_CORPORATE_ACTIONS = <número>

No iniciar backfill si existen acciones corporativas no resueltas que cambien
cantidades o símbolos.

==================================================
5. TRATAMIENTO DE REQUESTED_NOTIONAL
==================================================

A. CONOCIDOS

Para los 46 casos con log:

requested_notional = 1000
requested_notional_status = KNOWN
allocation_source = LOGGED_REQUEST
reconstruction_confidence = PROVEN

B. PRE-LOG

Para los 26 casos anteriores al log:

requested_notional = NULL
requested_notional_status = UNKNOWN_PRE_LOG
allocation_source = LEGACY_EQUAL_WEIGHT_RECONSTRUCTION
reconstruction_confidence = INFERRED
certification_state = MANUAL_REVIEW_REQUIRED

C. UNLINKED

Para los 10 casos sin vínculo:

requested_notional = NULL
requested_notional_status = UNKNOWN_UNLINKED
allocation_source según evidencia:
- LEGACY_EQUAL_WEIGHT_RECONSTRUCTION;
- CURRENT_HOLDING_STATE;
- UNKNOWN.

reconstruction_confidence:
- DETERMINISTIC;
- INFERRED;
- UNKNOWN.

certification_state =
MANUAL_REVIEW_REQUIRED

salvo evidencia adicional explícita que Code debe presentar antes de cambiar
el estado.

No utilizar como requested_notional:

- allocated_notional;
- capital/N;
- cantidad por precio;
- 1000 por analogía;
- otro registro.

La existencia de precio completo no cambia requested_notional_status.

==================================================
6. CONFIGURACIÓN HISTÓRICA POR CARTERA
==================================================

Crear una configuración histórica para cada una de las cinco carteras.

La fuente del capital debe ser:

carteras_papel.importe

No hardcodear 10000 aunque actualmente las cinco tengan ese valor.

Campos:

- paper_portfolio_id;
- methodology_id;
- configured_capital;
- currency;
- allocation_policy_id;
- partial_allocation_policy;
- leverage_allowed;
- effective_from;
- effective_to;
- status;
- source;
- configuration_hash.

Para la historia:

allocation_policy_id =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

status =
HISTORICAL_REPLAY

No activar MANUAL_NOTIONAL_HARD_CAP.

effective_from debe corresponder al inicio histórico reconstruible de cada
cartera, no a la fecha de migración.

Una configuración por cartera no constituye una aportación.

==================================================
7. APORTACIÓN INICIAL
==================================================

Crear exactamente una aportación inicial externa por cartera.

Importe:

initial_contribution =
configured_capital de esa cartera.

No hardcodear 10000.

Effective session:

- sesión inicial de la cartera según la primera composición económica;
- antes de aplicar compras internas de esa sesión.

Orden económico:

1. aportar capital;
2. emitir unidades a paper_unit_value = 100;
3. registrar paper cash;
4. ejecutar compras/rebalanceos internos;
5. mantener unit value neutral al flujo y a las compras.

Por cartera:

initial_units =
initial_contribution / 100

El flujo inicial debe ser:

flow_type =
PAPER_INITIAL_CONTRIBUTION

No crear otras aportaciones históricas salvo evidencia explícita.

Si el modelo legacy mantuvo siempre la suma asignada igual al capital, la
primera composición puede desplegar todo el cash.

No crear aportaciones adicionales por nuevas incorporaciones o reasignaciones.

==================================================
8. POSICIONES GLOBALES
==================================================

Crear 82 paper_global_position.

Identidad:

- source_holding_id;
- ticker;
- thesis_id;
- incorporated_at_utc;
- original lineage.

No depender únicamente del ticker.

Estado:

OPEN

porque no existen cierres económicos globales.

Conservar:

- timestamp original;
- ticker original;
- tesis;
- catalyst snapshot cuando exista;
- original quantity;
- original entry price;
- original notional;
- content hash;
- source references.

No modificar holdings legacy.

==================================================
9. MEMBERSHIP EPISODES
==================================================

Crear 121 paper_portfolio_membership_episode.

A. ABIERTOS

- 82 memberships actuales;
- status = OPEN.

B. CERRADOS

- 39 memberships en la cartera de origen;
- status = CLOSED;
- exit_reason = INTER_PORTFOLIO_REASSIGNMENT;
- global position continúa OPEN.

Cada reasignación debe:

- compartir reassignment_group_id;
- cerrar origen;
- abrir destino;
- utilizar el mismo timestamp efectivo;
- utilizar la misma sesión;
- utilizar el mismo cierre canónico;
- no crear flujo externo;
- no transferir unidades TWR entre carteras.

No duplicar el efecto económico.

==================================================
10. ALLOCATION REQUESTS
==================================================

Crear una allocation request por incorporación global histórica, salvo que el
modelo de lineage demuestre solicitudes adicionales independientes.

Resultado esperado:

paper_allocation_request = 82

Distribución:

- 46 KNOWN;
- 26 UNKNOWN_PRE_LOG;
- 10 UNKNOWN_UNLINKED.

Una reasignación no crea una nueva solicitud original de Omar salvo que el log
demuestre una solicitud distinta.

Puede crear una resolución/asignación nueva en destino vinculada a la misma
solicitud global.

No reinterpretar una reasignación como requested_notional nuevo.

==================================================
11. REPLAY DE COMPOSICIONES
==================================================

Reconstruir por cartera la política:

LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

Orden determinista:

1. timestamp absoluto;
2. log sequence/id;
3. regla de desempate documentada.

Para cada cambio:

- composición antes;
- composición después;
- configured capital;
- N antes;
- N después;
- allocated notional antes;
- allocated notional después;
- cantidad antes;
- cantidad después;
- paper cash antes;
- paper cash después;
- effective timestamp;
- effective session;
- source event;
- confidence;
- certification state.

Fórmula legacy:

allocated_notional =
configured_capital / active_membership_count

salvo evidencia de pesos explícitos distintos.

No utilizar la composición final retrospectivamente.

Ejemplo:

N=10 → 1000 por miembro.
N=11 → 909,09 por miembro desde esa sesión.
N=12 → 833,33 desde la siguiente versión efectiva correspondiente.

Las 39 reasignaciones deben producir:

- nueva composición de origen;
- nueva composición de destino;
- transacciones internas en ambas;
- cero flujo externo.

==================================================
12. CANTIDADES CERTIFICADAS
==================================================

Para cada composición y miembro:

certified_quantity_after =
allocated_notional_after
/
official_close de la effective_session

Conservar:

- original_quantity;
- original_entry_price;
- original_notional;
- certified_quantity_before;
- certified_quantity_after;
- allocated_notional_before;
- allocated_notional_after;
- price;
- price source;
- price basis;
- effective session.

Para una composición inicial:

- compra interna al cierre;
- P&L inicial = 0.

Para un rebalanceo:

- reconocer primero la variación hasta el cierre;
- después comprar o vender al cierre;
- la transacción no cambia unit value por sí misma;
- no reescribir sesiones anteriores.

No cambiar holdings legacy.

==================================================
13. TRANSACCIONES INTERNAS
==================================================

Crear las transacciones derivadas del replay.

Tipos autorizados:

- PURCHASE;
- SALE;
- LEGACY_REBALANCE_BUY;
- LEGACY_REBALANCE_SELL;
- SPLIT_ADJUSTMENT, cuando corresponda.

Para las reasignaciones, utilizar:

- SALE o LEGACY_REBALANCE_SELL en origen;
- PURCHASE o LEGACY_REBALANCE_BUY en destino;
- reason = INTER_PORTFOLIO_REASSIGNMENT;
- reassignment_group_id compartido.

No crear simultáneamente tipos TRANSFER y SALE/PURCHASE si duplican el efecto.

No crear:

- flujo externo por compra;
- flujo externo por venta;
- retirada por reasignación;
- aportación en destino.

Cada transacción debe contener:

- portfolio;
- membership;
- composition version;
- ticker;
- session;
- quantity delta;
- price;
- notional;
- cash delta;
- reason;
- idempotency key;
- content hash.

==================================================
14. PAPER CASH
==================================================

Reconstruir paper cash por cartera y versión.

Identidad:

paper_cash =
cumulative external contributions
- cumulative external withdrawals
- coste neto de compras
+ proceeds netos de ventas

En legacy equal-weight:

- la composición normalmente despliega todo el capital;
- pueden existir diferencias de redondeo;
- no permitir cash negativo;
- no crear aportaciones implícitas para cubrir diferencias.

Tolerancia de redondeo:

- explícita;
- documentada;
- consistente con almacenamiento monetario v27.

Si aparece cash negativo fuera de tolerancia:

- bloquear esa composición;
- no corregirla silenciosamente.

Entregar:

MIN_PAPER_CASH_BY_PORTFOLIO
MAX_NEGATIVE_ROUNDING_ADJUSTMENT
NEGATIVE_CASH_BLOCKED_CASES

==================================================
15. BENCHMARK INICIAL
==================================================

No crear todavía snapshots diarios del benchmark.

Sí persistir en configuración o flujo la referencia necesaria para Gate 2D:

- benchmark ticker = SPY;
- initial contribution session;
- initial external capital;
- validated SPY close;
- source;
- price basis.

No crear compras SPY por:

- incorporaciones internas;
- reasignaciones;
- rebalanceos legacy.

El benchmark replica únicamente los cinco flujos iniciales externos.

==================================================
16. PREVIEW FINAL ANTES DE ESCRIBIR
==================================================

Generar un artefacto final con todas las filas propuestas.

Contadores:

- configs;
- global positions;
- memberships;
- allocation requests;
- composition versions;
- composition members;
- flows;
- transactions;
- track snapshots.

Clasificar cada fila:

CERTIFIED

- evidencia suficiente;
- precio validado;
- policy demostrada;
- lineage resuelto.

MANUAL_REVIEW_REQUIRED

- requested desconocido;
- pre-log inferido;
- otra semántica no demostrada.

PRICE_COVERAGE_INCOMPLETE

- no esperado tras 2C-1;
- bloquear escritura del objeto afectado.

BLOCKED

- contradicción;
- cash negativo;
- acción corporativa no resuelta;
- lineage inconsistente.

No escribir si:

- existe cualquier BLOCKED;
- falta cualquier precio obligatorio;
- los contadores no reconcilian;
- las 39 reasignaciones no son atómicas;
- los memberships no suman 121;
- las posiciones globales no suman 82.

Puede escribirse información provisional con MANUAL_REVIEW_REQUIRED solo si:

- el schema preserva claramente su estado;
- no se presenta como certificada;
- no se utilizará en Gate 2D como serie certificada sin aprobación posterior.

==================================================
17. BACKFILL TRANSACCIONAL
==================================================

Antes:

- detener escrituras;
- una sola instancia;
- backup SQLite;
- SHA-256;
- abrir backup;
- integrity_check del backup;
- foreign_key_check;
- contadores;
- hashes;
- preview final.

Ejecutar todo el backfill dentro de una única transacción.

Orden:

1. configs;
2. global positions;
3. allocation requests;
4. membership episodes;
5. composition versions;
6. composition members;
7. initial flows;
8. internal transactions;
9. comprobaciones;
10. COMMIT.

paper_track_snapshot debe permanecer vacío.

Si falla:

- ROLLBACK completo;
- tablas operativas deben volver a cero;
- schema permanece v27;
- no continuar.

El backfill debe ser idempotente.

Una segunda ejecución:

- no duplica filas;
- no modifica hashes;
- no activa políticas;
- no crea snapshots.

==================================================
18. REVISIÓN MANUAL
==================================================

Después del backfill, generar una lista exacta de objetos:

- 26 UNKNOWN_PRE_LOG;
- 10 UNKNOWN_UNLINKED;
- cualquier composición pre-log inferida;
- cualquier otra fila MANUAL_REVIEW_REQUIRED.

Para cada uno mostrar:

- ticker;
- cartera;
- timestamp;
- requested_notional;
- allocated_notional candidato;
- cantidad certificada candidata;
- source;
- confidence;
- motivo;
- impacto sobre la futura serie;
- acción que debe aprobar Omar.

No convertirlos automáticamente en CERTIFIED.

No bloquear la persistencia provisional si:

- lineage está resuelto;
- precio está completo;
- estado manual queda explícito;
- Gate 2D excluye correctamente objetos no certificados.

==================================================
19. PRUEBAS
==================================================

Crear:

verify-paper-backfill-v27.mjs
verify-paper-membership-backfill.mjs
verify-paper-composition-replay.mjs
verify-paper-transaction-replay.mjs
verify-paper-cash-reconstruction.mjs
verify-paper-backfill-idempotency.mjs
verify-paper-backfill-isolation.mjs

Casos mínimos:

A. 82 global positions.
B. 121 memberships.
C. 39 memberships cerrados.
D. 39 reasignaciones atómicas.
E. 46 requested conocidos.
F. 26 unknown pre-log.
G. 10 unknown unlinked.
H. legacy N=10→11→12.
I. no composición final aplicada hacia atrás.
J. compra inicial P&L=0.
K. rebalanceo sin flujo externo.
L. reasignación sin flujo externo.
M. cash no negativo.
N. idempotencia.
O. track snapshots = 0.
P. hard cap sin activar.
Q. Crecimiento sin regresión.
R. aislamiento del patrimonio real.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

==================================================
20. PRODUCCIÓN DESPUÉS DEL BACKFILL
==================================================

Verificar:

- schema v27;
- integrity_check;
- foreign_key_check;
- una sola instancia;
- health;
- env-info;
- política hard cap sin effective_from;
- Track de Crecimiento idéntico;
- cash real idéntico;
- NAV real idéntico;
- holdings legacy intactos;
- snapshots legacy intactos;
- tesis intactas;
- catalizadores intactos;
- Campo de Caza intacto.

Contadores esperados mínimos:

paper_portfolio_track_config = 5
paper_global_position = 82
paper_portfolio_membership_episode = 121
paper_allocation_request = 82
paper_portfolio_flow = 5
paper_track_snapshot = 0

Los contadores de:

- paper_composition_version;
- paper_composition_member;
- paper_portfolio_transaction;

deben coincidir con el preview final y explicarse.

==================================================
21. CÓDIGO Y COMMIT
==================================================

Añadir únicamente:

- soporte de early close;
- tooling de replay;
- backfill v27;
- validadores;
- pruebas;
- documentación.

No añadir:

- staging JSON;
- scratchpad;
- backups;
- bases;
- datos productivos;
- artefactos de revisión.

Ejecutar secret scan.

Crear un único commit local:

feat(lens): backfill point-in-time paper portfolio history

No crear tag.
No hacer push.

==================================================
22. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = 4c97712
CANONICAL_HEAD_AFTER = <hash>

SCHEMA_BEFORE = v27
SCHEMA_AFTER = v27

EARLY_CLOSE_SUPPORT = PASS/FAIL
EARLY_CLOSE_EVENTS_IN_HISTORY = <número>
ANCHORS_CHANGED_BY_EARLY_CLOSE = <número>

CORPORATE_ACTIONS_CLASSIFIED = PASS/FAIL
UNRESOLVED_CORPORATE_ACTIONS = <número>

VALIDATED_PRICE_POINTS = 863
PRICE_COVERAGE = <porcentaje>

MEMBERSHIP_EPISODES_TOTAL = 121
MEMBERSHIP_EPISODES_PRICE_COMPLETE = <número>

PAPER_PORTFOLIO_CONFIGS_CREATED = <número>
PAPER_GLOBAL_POSITIONS_CREATED = <número>
PAPER_MEMBERSHIPS_CREATED = <número>
PAPER_OPEN_MEMBERSHIPS = <número>
PAPER_CLOSED_MEMBERSHIPS = <número>
PAPER_ALLOCATION_REQUESTS_CREATED = <número>

PAPER_COMPOSITION_VERSIONS_CREATED = <número>
PAPER_COMPOSITION_MEMBERS_CREATED = <número>
PAPER_INITIAL_FLOWS_CREATED = <número>
PAPER_TRANSACTIONS_CREATED = <número>
PAPER_TRACK_SNAPSHOTS_CREATED = 0

REQUESTED_KNOWN = 46
REQUESTED_UNKNOWN_PRE_LOG = 26
REQUESTED_UNKNOWN_UNLINKED = 10

CERTIFIED_OBJECTS = <número>
MANUAL_REVIEW_REQUIRED_OBJECTS = <número>
PRICE_COVERAGE_INCOMPLETE_OBJECTS = <número>
BLOCKED_OBJECTS = <número>

MIN_PAPER_CASH_BY_PORTFOLIO = <tabla>
NEGATIVE_CASH_BLOCKED_CASES = <número>

REASSIGNMENTS_ATOMIC = PASS/FAIL
LEGACY_REPLAY_AS_OPERATED = PASS/FAIL
FINAL_COMPOSITION_NOT_APPLIED_RETROACTIVELY = PASS/FAIL

BACKFILL_TRANSACTIONAL = PASS/FAIL
BACKFILL_IDEMPOTENT = PASS/FAIL

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

PRODUCTION_OPERATIONAL_ROWS_CREATED = <número>
NO_TRACK_SNAPSHOTS_CREATED = PASS/FAIL
NO_POLICY_ACTIVATED = PASS/FAIL
NO_GROWTH_TRACK_CHANGED = PASS/FAIL
NO_REAL_CASH_CHANGED = PASS/FAIL
NO_REAL_NAV_CHANGED = PASS/FAIL
NO_HOLDINGS_LEGACY_CHANGED = PASS/FAIL
NO_SNAPSHOTS_LEGACY_CHANGED = PASS/FAIL
NO_THESES_CHANGED = PASS/FAIL
NO_CATALYSTS_CHANGED = PASS/FAIL
NO_HUNTING_FIELD_CHANGED = PASS/FAIL
NO_REAL_ECONOMIC_WRITES = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

LOCAL_COMMIT = <hash>

EXACT_MANUAL_APPROVALS_REQUIRED_BEFORE_GATE_2D:
1. <aprobación>
2. <aprobación>
3. <aprobación>

Detenerse después del Gate 2C-2.