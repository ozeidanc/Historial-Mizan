MIZAN · LA LENTE
GATE 2C-2 · BACKFILL HISTÓRICO POINT-IN-TIME
COMPOSICIONES EOD NETAS · HISTORIA AS-OPERATED · SIN SERIE DIARIA

BASELINE

CANONICAL_HEAD esperado = 4c97712
schema = v27
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

GATE 2C-1 ACEPTADO

Cobertura disponible en staging:

- 82/82 anchors;
- 39/39 reasignaciones;
- 812/812 precios diarios requeridos;
- 15/15 sesiones SPY;
- 863 puntos únicos validados;
- 100 % de cobertura;
- ventana 2026-07-06 a 2026-07-24.

MISIÓN

1. Soportar cierres anticipados.
2. Validar y sellar el manifiesto de precios utilizado.
3. Clasificar las acciones corporativas.
4. Clasificar los diez UNKNOWN_UNLINKED.
5. Reconstruir el histórico AS_OPERATED.
6. Poblar las tablas estructurales de v27.
7. Versionar todas las composiciones históricas.
8. Aplicar económicamente solo la composición EOD final de cada sesión.
9. Crear transacciones netas, sin turnover artificial.
10. Reconciliar el estado final con el legado.
11. Mantener paper_track_snapshot vacío.
12. Mantener inactiva MANUAL_NOTIONAL_HARD_CAP.

No modificar todavía:

- daily-close;
- startup catch-up;
- endpoints del Track Papel;
- UI;
- TrackRecordView;
- política operativa futura.

==================================================
0. RESTRICCIONES
==================================================

No modificar:

- holdings legacy;
- snapshots legacy;
- cat_composicion_log;
- tesis;
- catalizadores;
- Campo de Caza;
- Track de Crecimiento;
- Book real;
- cash real;
- NAV real;
- movimientos;
- decisiones;
- resizing;
- valuations reales.

No hacer push.
No crear tag.
No realizar operaciones Git remotas.
No conectar con Wio.
No crear operaciones reales.

La política prospectiva debe permanecer:

policy_id =
MANUAL_NOTIONAL_HARD_CAP

status =
APPROVED_PENDING_ACTIVATION

effective_from =
NULL

Las únicas escrituras autorizadas son el backfill estructural v27 de La
Lente.

paper_track_snapshot debe permanecer vacío.

==================================================
1. IDENTIFICADORES CANÓNICOS
==================================================

Utilizar exactamente los identificadores existentes en v27.

TRACK METHODOLOGY:

PAPER_PRICE_RETURN_TWR_V1

HISTORICAL ALLOCATION POLICY:

LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

PROSPECTIVE ALLOCATION POLICY:

MANUAL_NOTIONAL_HARD_CAP

No introducir identificadores alternativos como:

- PAPER_LEGACY_EQUAL_WEIGHT_AS_OPERATED_V1;
- PAPER_MANUAL_NOTIONAL_HARD_CAP_V1;

salvo que realmente existan en v27 y Code demuestre su relación versionada.

Antes del backfill verificar:

- una metodología sellada;
- tres políticas globales esperadas;
- legacy disponible solo para replay;
- hard cap pendiente de activación;
- effective_from de hard cap = NULL.

==================================================
2. PRECONDICIONES
==================================================

Antes de escribir:

- confirmar HEAD;
- confirmar env-info;
- confirmar schema v27;
- confirmar una sola raíz Git;
- inventariar archivos untracked;
- confirmar una sola instancia;
- detener escrituras de aplicación;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- validar las definiciones de metodología y políticas;
- comprobar hashes de los artefactos de Gate 2C-1;
- comprobar que el staging no fue alterado;
- confirmar cobertura de precios del 100 %;
- confirmar que las nueve tablas operativas están vacías.

Tablas operativas:

1. paper_portfolio_track_config;
2. paper_global_position;
3. paper_portfolio_membership_episode;
4. paper_allocation_request;
5. paper_composition_version;
6. paper_composition_member;
7. paper_portfolio_flow;
8. paper_portfolio_transaction;
9. paper_track_snapshot.

Capturar antes:

- contadores reales;
- contadores legacy;
- hashes de holdings;
- hashes de snapshots;
- hashes de cat_composicion_log;
- hash de valuations de Crecimiento;
- payload de /track/crecimiento;
- métricas;
- ventanas;
- benchmark;
- drawdown;
- cash real;
- NAV real.

No continuar si:

- alguna tabla operativa no está vacía;
- staging no coincide con sus hashes;
- falta un precio obligatorio;
- la política hard cap está activa;
- integrity_check o foreign_key_check falla.

==================================================
3. CALENDARIO Y CIERRES ANTICIPADOS
==================================================

El calendario debe exponer por sesión:

- session_date;
- session_open_at;
- session_close_at;
- session_type:
  - REGULAR;
  - EARLY_CLOSE;
  - CLOSED;
- timezone = America/New_York.

No asumir siempre:

session_close_at = 16:00

Probar:

- sesión regular;
- Black Friday;
- sesión previa a Independence Day cuando corresponda;
- pre-market;
- post-market;
- DST;
- festivo;
- fin de semana.

Para early close:

- antes del cierre → sesión abierta;
- después del cierre → sesión cerrada;
- anchor = cierre oficial de esa sesión.

Verificar el histórico actual.

Entregar:

EARLY_CLOSE_SUPPORTED = PASS/FAIL
EARLY_CLOSE_EVENTS_IN_HISTORY = <número>
ANCHORS_CHANGED_BY_EARLY_CLOSE = <número>
REASSIGNMENTS_CHANGED_BY_EARLY_CLOSE = <número>

No continuar si alguna sesión histórica cambia y el manifiesto no se
recalcula.

==================================================
4. MANIFIESTO INMUTABLE DE PRECIOS
==================================================

Crear un manifiesto sellado de los 863 puntos validados.

Cada punto debe contener:

- price_evidence_id;
- requested_ticker;
- canonical_ticker;
- provider_symbol;
- valuation_session;
- purposes;
- provider;
- price_basis;
- nominal_close;
- adjusted_close informativo;
- currency;
- fetched_at;
- source_identifier;
- source_payload_hash;
- manifest_hash;
- validation_status.

Purposes:

- ANCHOR;
- DAILY_VALUATION;
- MEMBERSHIP_REASSIGNMENT;
- LEGACY_REBALANCE;
- BENCHMARK.

No persistir los 863 puntos como paper_track_snapshot.

Pero toda fila económica creada debe poder referenciar:

- manifest_hash;
- provider;
- valuation_session;
- price_basis;
- source_payload_hash o price_evidence_id.

Los artefactos JSON de scratchpad no deben ser la única evidencia de los
precios utilizados.

Si v27 no dispone de una tabla específica de evidencia:

- conservar los campos de evidencia en memberships, composiciones y
  transacciones;
- documentar cómo Gate 2D reproducirá exactamente los precios;
- no depender únicamente de volver a consultar al proveedor.

Declarar:

PRICE_EVIDENCE_POINTS = 863
PRICE_MANIFEST_VALID = PASS/FAIL
PRICE_MANIFEST_HASH = <hash>
PRICE_EVIDENCE_DURABLY_REFERENCED = PASS/FAIL
PAPER_TRACK_SNAPSHOTS_CREATED = 0

==================================================
5. COBERTURA DE LOS 121 MEMBERSHIPS
==================================================

Validar cobertura por membership episode.

Para cada uno:

- global position;
- portfolio;
- entered_at;
- entered_session;
- exited_at;
- exited_session;
- first required session;
- last required session;
- required points;
- validated points;
- coverage status.

Resultado obligatorio:

MEMBERSHIP_EPISODES_TOTAL = 121
MEMBERSHIP_EPISODES_PRICE_COMPLETE = 121
OPEN_MEMBERSHIPS_PRICE_COMPLETE = 82
CLOSED_MEMBERSHIPS_PRICE_COMPLETE = 39

No ejecutar backfill si no se cumplen los cuatro valores.

==================================================
6. SPY
==================================================

Reconciliar:

SPY_REQUIRED_SESSIONS = 15
SPY_RETURNED_POINTS = <número>
SPY_USED_REQUIRED_SESSIONS = 15

Excluir de la metodología:

- puntos anteriores a la inception de cada cartera;
- sesiones no requeridas;
- puntos posteriores a coverage_end_session.

Los puntos adicionales pueden conservarse como respuesta de proveedor, pero
no utilizarse para retorno.

Declarar:

SPY_EXTRA_POINTS_EXCLUDED = PASS/FAIL

==================================================
7. ACCIONES CORPORATIVAS
==================================================

Clasificar DOX, IMMR y cualquier otra señal.

Por señal:

- ticker;
- fecha;
- event type;
- dentro o fuera de ventana;
- split ratio;
- dividend amount;
- symbol change;
- efecto sobre cierre;
- efecto sobre cantidad;
- efecto sobre PRICE_RETURN_TWR;
- tratamiento;
- source;
- blocking status.

DIVIDENDO:

- no añadir ingreso;
- no cambiar quantity;
- usar cierre nominal;
- registrar INCOME_NOT_INCLUDED;
- aceptar el efecto ex-dividendo sobre PRICE_RETURN_TWR.

SPLIT:

- ajustar quantity;
- preservar valor económico;
- crear SPLIT_ADJUSTMENT;
- no generar retorno artificial.

ACCIÓN NO IDENTIFICADA:

- BLOCKED;
- no ejecutar backfill del objeto afectado.

Declarar:

CORPORATE_ACTIONS_CLASSIFIED = PASS/FAIL
UNRESOLVED_CORPORATE_ACTIONS = <número>
SPLIT_CONTINUITY_VALIDATED = PASS/FAIL/N/A
DIVIDEND_EXCLUSION_VALIDATED = PASS/FAIL/N/A

No continuar si existe una acción no resuelta que afecte precios, símbolos o
cantidades.

==================================================
8. CLASIFICACIÓN DE UNKNOWN_UNLINKED
==================================================

Clasificar individualmente los diez registros:

- DIRECT_ASSIGNMENT;
- TECHNICAL_CREATION;
- MISSING_EVENT;
- MIGRATED_RECORD;
- ORPHANED_LINK;
- OTHER_EXACT_CAUSE.

Por registro:

- source_holding_id;
- ticker;
- portfolio;
- entrada_ts;
- thesis_id;
- assignment event;
- snapshot;
- causa;
- confidence;
- requested_notional = NULL;
- requested_notional_status = UNKNOWN_UNLINKED.

Resultado:

UNLINKED_RECORDS_CLASSIFIED = 10/10
UNRESOLVED_LINEAGE = 0

La clasificación no convierte requested_notional en conocido.

Para esos diez:

certification_state =
MANUAL_REVIEW_REQUIRED

salvo evidencia documental explícita del notional solicitado.

==================================================
9. REQUESTED NOTIONAL Y CONFIANZA
==================================================

A. 46 CONOCIDOS

requested_notional = 1000
requested_notional_status = KNOWN
allocation_source = LOGGED_REQUEST
reconstruction_confidence = PROVEN

B. 26 PRE-LOG

requested_notional = NULL
requested_notional_status = UNKNOWN_PRE_LOG
allocation_source = LEGACY_EQUAL_WEIGHT_RECONSTRUCTION
reconstruction_confidence = INFERRED
certification_state = MANUAL_REVIEW_REQUIRED

C. 10 UNLINKED

requested_notional = NULL
requested_notional_status = UNKNOWN_UNLINKED
allocation_source según evidencia
reconstruction_confidence según evidencia
certification_state = MANUAL_REVIEW_REQUIRED

DECISIÓN APROBADA PARA RECONSTRUCCIÓN:

Los 26 pre-log pueden persistirse como reconstrucción histórica candidata
AS_OPERATED usando:

- entrada_ts;
- capital configurado;
- memberships activos;
- fórmula legacy capital/N;
- cierre oficial.

Pero deben permanecer:

- INFERRED;
- MANUAL_REVIEW_REQUIRED;
- no presentados como requested_notional conocido;
- no denominados CERTIFIED.

Gate 2D podrá incluirlos únicamente con un estado visible de:

PARTIAL_HISTORICAL_CONFIDENCE

hasta aprobación manual posterior.

==================================================
10. CONFIGURACIONES POR CARTERA
==================================================

Crear cinco configuraciones históricas.

El capital debe leerse de:

carteras_papel.importe

No hardcodear 10.000.

Para cada cartera:

- paper_portfolio_id;
- methodology_id = PAPER_PRICE_RETURN_TWR_V1;
- configured_capital;
- currency;
- allocation_policy_id =
  LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE;
- leverage_allowed = false;
- effective_from;
- effective_to;
- status = HISTORICAL_REPLAY;
- source;
- configuration_hash.

No activar MANUAL_NOTIONAL_HARD_CAP.

Una configuración no constituye un flujo externo.

Comprobar que actualmente:

PAPER_INITIAL_CAPITAL_TOTAL =
suma real de carteras_papel.importe

No imponer 50.000 si los datos no lo confirman.

==================================================
11. FLUJOS INICIALES
==================================================

Crear una aportación inicial por cartera.

Importe:

initial_contribution =
configured_capital leído de la cartera.

Effective session:

primera sesión de composición económica de esa cartera.

Orden:

1. aportación externa;
2. emisión de unidades a unit value 100;
3. paper cash;
4. composición EOD;
5. compras internas netas;
6. unit value permanece 100 por el flujo y las compras.

No crear:

- flujo por incorporación;
- flujo por reasignación;
- aportación implícita;
- retirada por cambio de membership.

Resultado esperado:

PAPER_INITIAL_CONTRIBUTIONS = 5
PAPER_INITIAL_CAPITAL_TOTAL = <suma demostrada>

==================================================
12. POSICIONES Y MEMBERSHIPS
==================================================

Crear:

paper_global_position = 82

Estados:

- 82 OPEN;
- 0 económicamente cerradas.

Crear:

paper_portfolio_membership_episode = 121

Distribución:

- 82 OPEN;
- 39 CLOSED por INTER_PORTFOLIO_REASSIGNMENT.

Cada reasignación:

- comparte reassignment_group_id;
- cierra origen;
- abre destino;
- usa mismo instante;
- usa misma sesión;
- usa mismo cierre;
- no genera flujo externo;
- no transfiere unidades TWR.

No crear un requested_notional nuevo por una reasignación salvo evidencia de
una nueva solicitud de Omar.

==================================================
13. ALLOCATION REQUESTS
==================================================

Crear 82 solicitudes históricas:

- 46 KNOWN;
- 26 UNKNOWN_PRE_LOG;
- 10 UNKNOWN_UNLINKED.

Una allocation request puede tener requested_notional NULL.

Una reasignación utiliza la solicitud global existente.

No duplicar requests por:

- origen;
- destino;
- composición;
- snapshot.

==================================================
14. VERSIONES DE COMPOSICIÓN
==================================================

Reconstruir todas las versiones de auditoría en orden:

1. timestamp absoluto;
2. sequence/id;
3. regla de desempate estable.

Cada cambio de membership produce una versión de auditoría con:

- effective_at_utc;
- effective_session;
- event_sequence;
- triggering_event;
- source_log_event_id;
- members;
- N before;
- N after;
- allocated notional;
- confidence;
- certification state.

Fórmula legacy:

allocated_notional =
configured_capital / active_membership_count

salvo evidencia de pesos explícitos.

No aplicar la composición final retrospectivamente.

==================================================
15. EVENTOS MÚLTIPLES EN UNA MISMA SESIÓN
==================================================

Separar:

A. AUDIT COMPOSITION VERSION

Conserva cada evento y su orden intradía.

B. ECONOMIC EOD COMPOSITION

Es la última composición efectiva de cada cartera en la sesión.

Todos los eventos REGULAR_OPEN de una misma sesión utilizan el mismo cierre
oficial.

Por tanto:

- las versiones intermedias se conservan para auditoría;
- no generan una cadena artificial de compras y ventas al mismo cierre;
- la transición económica se calcula entre:
  - composición EOD de la sesión anterior;
  - composición EOD final de la sesión actual.

Las transacciones deben netearse por:

- paper_portfolio_id;
- ticker;
- effective_session;
- price_basis.

Fórmula conceptual:

net_quantity_delta =
final_eod_quantity
- previous_eod_quantity

Crear:

- BUY si net_quantity_delta > 0;
- SELL si net_quantity_delta < 0;
- ninguna transacción si net_quantity_delta = 0 dentro de tolerancia.

No crear transacciones brutas para cada versión intermedia si se cancelan
económicamente al mismo cierre.

No generar turnover artificial.

Declarar:

AUDIT_COMPOSITION_VERSIONS_CREATED = <número>
ECONOMIC_EOD_COMPOSITIONS_CREATED = <número>
SAME_SESSION_EVENTS_NETTED = PASS/FAIL
ARTIFICIAL_TURNOVER_CREATED = 0

==================================================
16. CANTIDADES CERTIFICADAS
==================================================

Para cada composición EOD:

certified_quantity =
allocated_notional
/
official_close de la sesión efectiva.

Conservar:

- original_quantity;
- original_entry_price;
- original_notional;
- requested_notional;
- requested_notional_status;
- allocated_notional;
- certified_quantity;
- official_close;
- price evidence;
- reconstruction confidence;
- certification state.

Para la composición inicial:

- compra neta al cierre;
- P&L inicial = 0.

Para un rebalanceo:

1. reconocer variación hasta el cierre;
2. calcular composición EOD final;
3. generar delta neto;
4. comprar o vender al cierre;
5. no modificar unit value por la transacción;
6. no reescribir sesiones anteriores.

==================================================
17. TRANSACCIONES INTERNAS NETAS
==================================================

Tipos económicos:

- PURCHASE;
- SALE;
- LEGACY_REBALANCE_BUY;
- LEGACY_REBALANCE_SELL;
- SPLIT_ADJUSTMENT.

Una reasignación se representa mediante:

- reason = INTER_PORTFOLIO_REASSIGNMENT;
- reassignment_group_id;
- transacción neta de salida en origen;
- transacción neta de entrada en destino.

No crear simultáneamente:

- REASSIGNMENT_OUT y SALE;
- REASSIGNMENT_IN y PURCHASE;

si producen el mismo efecto económico.

Las transacciones deben ser netas por cartera, ticker y sesión.

Cada transacción contiene:

- portfolio;
- global position;
- membership;
- economic EOD composition;
- session;
- quantity delta;
- price;
- notional;
- cash delta;
- reason;
- reassignment group;
- price evidence reference;
- idempotency key;
- content hash.

No crear transacciones de importe cero.

==================================================
18. PAPER CASH Y CAPITAL
==================================================

Por cada composición EOD:

paper_cash =
cumulative external contributions
- cumulative external withdrawals
- net purchases
+ net sales

Identidad:

configured_capital =
paper_cash
+ allocated_notional total

sujeta únicamente a tolerancia de redondeo documentada.

No permitir:

- cash negativo fuera de tolerancia;
- leverage;
- aportación implícita;
- corrección silenciosa.

Si existe cash negativo:

- BLOCKED;
- rollback completo.

Entregar:

MIN_PAPER_CASH_BY_PORTFOLIO
MAX_ROUNDING_DIFFERENCE
NEGATIVE_CASH_BLOCKED_CASES

==================================================
19. BENCHMARK
==================================================

No crear todavía snapshots diarios SPY.

Persistir referencia de cada flujo inicial:

- portfolio;
- inception session;
- initial contribution;
- SPY close;
- provider;
- price basis;
- manifest hash.

SPY replica únicamente flujos externos.

No crear operaciones SPY por:

- incorporaciones;
- reasignaciones;
- rebalanceos;
- cambios de membership.

==================================================
20. PREVIEW FINAL Y GATE DE ESCRITURA
==================================================

Antes de escribir, generar todas las filas propuestas.

Contadores:

- configs;
- global positions;
- memberships;
- allocation requests;
- audit composition versions;
- economic EOD compositions;
- composition members;
- flows;
- net transactions;
- track snapshots.

Clasificación:

CERTIFIED

- evidencia suficiente;
- precio completo;
- lineage resuelto;
- semántica demostrada.

MANUAL_REVIEW_REQUIRED

- 26 pre-log;
- 10 unlinked;
- requested desconocido;
- reconstrucción inferida.

BLOCKED

- contradicción;
- precio faltante;
- acción corporativa no resuelta;
- cash negativo;
- lineage inconsistente;
- reconciliación final fallida.

No escribir si existe cualquier BLOCKED.

Se permite persistir MANUAL_REVIEW_REQUIRED porque:

- queda etiquetado;
- no se presenta como certificado;
- Gate 2D deberá distinguirlo;
- el dato original no se modifica.

==================================================
21. RECONCILIACIÓN FINAL
==================================================

Comparar el último estado EOD reconstruido con holdings legacy.

Por cartera:

- tickers;
- memberships;
- allocated notional;
- peso;
- capital;
- exposición.

No exigir igualdad entre:

- legacy quantity;
- certified quantity.

Sí exigir:

REPLAY_FINAL_MEMBERSHIP_MATCH = PASS
REPLAY_FINAL_ALLOCATED_NOTIONAL_MATCH = PASS
REPLAY_FINAL_WEIGHT_MATCH = PASS
PAPER_CAPITAL_IDENTITY = PASS

Aplicar tolerancia monetaria explícita.

Si falla:

- no escribir;
- o rollback completo si el fallo aparece durante validación transaccional.

==================================================
22. BACKFILL TRANSACCIONAL
==================================================

Antes:

- backup;
- SHA-256;
- abrir backup;
- integrity_check;
- foreign_key_check;
- contadores;
- hashes;
- preview final.

Ejecutar en una única transacción:

1. configs;
2. global positions;
3. allocation requests;
4. memberships;
5. audit composition versions;
6. economic EOD compositions, si el schema las distingue;
7. composition members;
8. initial flows;
9. net internal transactions;
10. validations;
11. COMMIT.

paper_track_snapshot permanece vacío.

Si falla:

- ROLLBACK;
- tablas operativas vuelven a cero;
- schema permanece v27.

Segunda ejecución:

- devuelve ALREADY_BACKFILLED;
- no duplica;
- no actualiza;
- no cambia hashes;
- no crea snapshots;
- no activa políticas.

==================================================
23. PRUEBAS
==================================================

Crear o consolidar:

verify-paper-calendar-early-close.mjs
verify-paper-price-manifest.mjs
verify-paper-corporate-actions.mjs
verify-paper-backfill-2c2.mjs
verify-paper-membership-backfill.mjs
verify-paper-composition-replay.mjs
verify-paper-same-session-netting.mjs
verify-paper-reassignment.mjs
verify-paper-capital-identity.mjs
verify-paper-backfill-idempotency.mjs
verify-paper-backfill-isolation.mjs

Casos obligatorios:

- 82 global positions;
- 121 memberships;
- 39 memberships cerrados;
- 39 reasignaciones atómicas;
- 46 requested conocidos;
- 26 unknown pre-log;
- 10 unknown unlinked;
- legacy N=10→11→12;
- múltiples eventos misma sesión;
- cero turnover artificial;
- no composición final hacia atrás;
- compra inicial P&L 0;
- rebalanceo sin flujo;
- reasignación sin flujo;
- cash no negativo;
- idempotencia;
- snapshots = 0;
- hard cap sin activar;
- Crecimiento sin regresión;
- aislamiento real.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

==================================================
24. PRODUCCIÓN
==================================================

Después:

- schema v27;
- integrity_check;
- foreign_key_check;
- una sola instancia;
- health;
- env-info;
- hard cap sin effective_from;
- paper_track_snapshot = 0.

Confirmar idénticos:

- holdings legacy;
- snapshots legacy;
- cat_composicion_log;
- tesis;
- catalizadores;
- Campo de Caza;
- Book real;
- cash real;
- NAV real;
- Track Crecimiento.

==================================================
25. CÓDIGO Y COMMIT
==================================================

Añadir únicamente:

- soporte de early close;
- manifiesto/referencias de precios;
- replay;
- netting EOD;
- backfill;
- validadores;
- pruebas;
- documentación.

No añadir:

- scratchpad;
- staging JSON;
- backups;
- bases;
- datos productivos.

Ejecutar secret scan.

Crear un commit local:

feat(lens): backfill point-in-time paper portfolio history

No crear tag.

El texto:

mizan-lens-paper-track-backfill-v1.0

queda reservado como posible tag futuro, no como mensaje de commit.

No hacer push.

==================================================
26. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = 4c97712
CANONICAL_HEAD_AFTER = <hash>

EARLY_CLOSE_SUPPORTED = PASS/FAIL
EARLY_CLOSE_EVENTS_IN_HISTORY = <número>
ANCHORS_CHANGED_BY_EARLY_CLOSE = <número>
REASSIGNMENTS_CHANGED_BY_EARLY_CLOSE = <número>

PRICE_EVIDENCE_POINTS = 863
PRICE_MANIFEST_VALID = PASS/FAIL
PRICE_MANIFEST_HASH = <hash>
PRICE_EVIDENCE_DURABLY_REFERENCED = PASS/FAIL

MEMBERSHIP_EPISODES_TOTAL = 121
MEMBERSHIP_EPISODES_PRICE_COMPLETE = <número>

CORPORATE_ACTIONS_CLASSIFIED = PASS/FAIL
UNRESOLVED_CORPORATE_ACTIONS = <número>

UNLINKED_RECORDS_CLASSIFIED = <número>/10
UNRESOLVED_LINEAGE = <número>

PAPER_CONFIGS_CREATED = <número>/5
PAPER_INITIAL_CONTRIBUTIONS = <número>/5
PAPER_GLOBAL_POSITIONS_CREATED = <número>/82
PAPER_OPEN_MEMBERSHIPS_CREATED = <número>/82
PAPER_CLOSED_MEMBERSHIPS_CREATED = <número>/39
PAPER_TOTAL_MEMBERSHIPS_CREATED = <número>/121
PAPER_ALLOCATION_REQUESTS_CREATED = <número>/82

AUDIT_COMPOSITION_VERSIONS_CREATED = <número>
ECONOMIC_EOD_COMPOSITIONS_CREATED = <número>
COMPOSITION_MEMBERS_CREATED = <número>
NET_INTERNAL_TRANSACTIONS_CREATED = <número>

REQUESTED_NOTIONAL_KNOWN = 46
REQUESTED_NOTIONAL_UNKNOWN_PRE_LOG = 26
REQUESTED_NOTIONAL_UNKNOWN_UNLINKED = 10

CERTIFIED_OBJECTS = <número>
MANUAL_REVIEW_REQUIRED_OBJECTS = <número>
BLOCKED_OBJECTS = <número>

SAME_SESSION_EVENTS_NETTED = PASS/FAIL
ARTIFICIAL_TURNOVER_CREATED = <número>

REPLAY_FINAL_MEMBERSHIP_MATCH = PASS/FAIL
REPLAY_FINAL_ALLOCATED_NOTIONAL_MATCH = PASS/FAIL
REPLAY_FINAL_WEIGHT_MATCH = PASS/FAIL
PAPER_CAPITAL_IDENTITY = PASS/FAIL

MIN_PAPER_CASH_BY_PORTFOLIO = <tabla>
NEGATIVE_CASH_BLOCKED_CASES = <número>

BACKFILL_TRANSACTIONAL = PASS/FAIL
BACKFILL_IDEMPOTENT = PASS/FAIL

PAPER_TRACK_SNAPSHOTS_CREATED = 0

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

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