MIZAN · LA LENTE
GATE 2D-2 · MOTOR DIARIO DEL TRACK RECORD PAPEL
SNAPSHOTS PRICE-RETURN TWR · BENCHMARK SPY · DAILY-CLOSE · ENDPOINTS

NATURALEZA

Esta ejecución autoriza:

1. construir la serie diaria del Track Papel;
2. marcar a mercado todas las sesiones aplicables;
3. crear paper_track_snapshot;
4. construir el benchmark SPY por cartera;
5. persistir el backfill diario de snapshots;
6. integrar un adaptador papel en daily-close;
7. implementar startup catch-up;
8. crear endpoints canónicos read-only;
9. generar métricas y read models;
10. desplegar el motor y los endpoints;
11. crear un único commit local.

Esta ejecución NO autoriza:

- modificar la presentación visual;
- crear o activar TrackRecordView;
- eliminar papelChartHTML;
- activar MANUAL_NOTIONAL_HARD_CAP;
- fijar effective_from de la política prospectiva;
- modificar la UI de incorporación;
- crear solicitudes prospectivas;
- modificar holdings legacy;
- modificar snapshots legacy;
- modificar cat_composicion_log;
- modificar tesis;
- modificar catalizadores;
- modificar Campo de Caza;
- cambiar el Track de Crecimiento;
- hacer push;
- crear tag.

La paridad visual corresponde a Gate 2E.

La activación operativa de hard cap corresponde a Gate 2F.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = f124193
schema esperado = v28
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Inventario esperado:

PAPER_PORTFOLIO_CONFIGS = 5
PAPER_INITIAL_FLOWS = 5
PAPER_GLOBAL_POSITIONS = 82
PAPER_MEMBERSHIP_EPISODES = 121
PAPER_OPEN_MEMBERSHIPS = 82
PAPER_CLOSED_MEMBERSHIPS = 39

PAPER_EOD_COMPOSITIONS = 19
PAPER_COMPOSITION_MEMBERS = 249

LEGACY_TRANSACTIONS = 249
CANONICAL_ECONOMIC_TRANSACTIONS = 249
ECONOMIC_RECONSTRUCTION_VERSION = 1

PAPER_ECONOMIC_STATES = 19
PAPER_TRACK_SNAPSHOTS = 0

PRICE_EVIDENCE_POINTS = 863

PRICE_MANIFEST_HASH =
3c5bf06f68db1021a491dc511470eb1a29b0c8be81bf53165ee3c1ab21cc943a

Metodología:

PAPER_TRACK_METHODOLOGY =
PAPER_PRICE_RETURN_TWR_V1

RETURN_METHOD =
PRICE_RETURN_TWR

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

COMMISSION_POLICY =
NOT_MODELED

EXIT_POLICY =
KEEP_AS_PAPER_CASH

BENCHMARK =
SPY

BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

Política histórica:

HISTORICAL_TRACK_POLICY =
AS_OPERATED

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

Política prospectiva:

PROSPECTIVE_ALLOCATION_POLICY =
MANUAL_NOTIONAL_HARD_CAP

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

No activar la política prospectiva.

==================================================
1. FUENTE ECONÓMICA CANÓNICA
==================================================

El motor debe utilizar exclusivamente:

- reconstruction_version = 1;
- status = CANONICAL;
- metodología NAV_CONSERVING;
- transacciones económicas canónicas;
- estados económicos canónicos;
- composiciones EOD;
- flujos externos;
- evidencia de precios validada.

No mezclar en un mismo cálculo:

- transacciones legacy de Gate 2C-2;
- transacciones canónicas de Gate 2D-1.

Las 249 transacciones legacy permanecen como:

LEGACY_NOTIONAL_AUDIT_ONLY

Las 249 transacciones económicas nuevas son las únicas autorizadas para el
Track.

Crear una prueba que falle si:

- existe más de una reconstrucción canónica;
- una consulta mezcla versiones;
- una transacción legacy entra en el cálculo;
- falta una transacción canónica requerida.

Declarar:

SINGLE_CANONICAL_RECONSTRUCTION = PASS/FAIL
LEGACY_TRANSACTIONS_EXCLUDED_FROM_TRACK = PASS/FAIL

==================================================
2. PRECONDICIONES
==================================================

Antes de modificar:

- confirmar HEAD f124193;
- confirmar env-info;
- confirmar schema v28;
- confirmar una única raíz Git;
- inventariar archivos untracked;
- confirmar una sola instancia;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- confirmar health;
- validar el manifiesto de precios y su hash;
- validar los contadores v27/v28;
- validar reconstruction_version = 1;
- validar NAV continuity;
- validar unit continuity;
- validar que paper_track_snapshot sigue vacío;
- validar hard cap sin effective_from.

Capturar baseline de Crecimiento:

- payload de /track/crecimiento;
- serie;
- fechas;
- métricas;
- ventanas;
- benchmark;
- drawdown;
- coverage;
- valuations;
- holdings;
- movimientos;
- cash;
- NAV;
- Book;
- hashes.

Capturar baseline de Papel:

- endpoint legacy;
- payload legacy;
- estado de gráfica;
- contadores;
- composiciones;
- transacciones;
- estados económicos.

No continuar si:

- hay más de una reconstrucción canónica;
- falta alguna transacción económica;
- el manifiesto no coincide;
- existe una acción corporativa bloqueada;
- hard cap está activo;
- integrity_check falla;
- foreign_key_check devuelve violaciones.

==================================================
3. COBERTURA FINAL DINÁMICA
==================================================

Resolver al comenzar:

execution_time_utc
execution_time_dubai
execution_time_new_york

Resolver:

coverage_end_session =
última sesión NYSE completamente cerrada cuyo precio EOD esté disponible y
validado.

No codificar permanentemente:

2026-07-24

Regla:

- una sesión abierta no entra;
- una sesión cerrada sin precio EOD validado no entra;
- no usar precio live;
- no usar previous close como sustituto;
- no usar fecha UTC directamente;
- no usar la fecha de Dubái como sesión económica.

Si coverage_end_session es posterior al manifiesto existente:

1. obtener precios mediante el proveedor canónico;
2. aplicar el mismo contrato de validación;
3. crear evidencia durable;
4. extender el manifiesto de forma versionada;
5. no modificar el manifiesto anterior in-place;
6. registrar nuevo hash.

Entregar:

ORIGINAL_MANIFEST_END_SESSION = 2026-07-24
CURRENT_COVERAGE_END_SESSION = <fecha>
PRICE_MANIFEST_VERSION = <versión>
CURRENT_PRICE_MANIFEST_HASH = <hash>

==================================================
4. SESIONES DIARIAS POR CARTERA
==================================================

Para cada cartera determinar:

first_valuation_session =
sesión de su aportación inicial y primera composición económica.

last_valuation_session =
coverage_end_session.

Generar todas las sesiones NYSE comprendidas, inclusive.

Respetar:

- sesiones regulares;
- early close;
- festivos;
- fines de semana;
- DST;
- America/New_York.

No generar snapshots en:

- días naturales sin sesión;
- sesiones todavía abiertas;
- sesiones sin evidencia de precios suficiente.

Antes de escribir, entregar por cartera:

- first session;
- last session;
- expected session count;
- covered session count;
- missing sessions;
- blocked sessions.

No persistir si alguna sesión obligatoria está bloqueada.

==================================================
5. PROPAGACIÓN DEL ESTADO ECONÓMICO
==================================================

En una sesión sin cambio de composición:

- conservar quantities de la última composición económica EOD;
- conservar units;
- conservar paper cash;
- marcar a mercado con los cierres actuales;
- no crear transacciones;
- no crear flujos;
- calcular NAV, unit value y retorno.

En una sesión con cambio de composición:

1. cargar quantities y cash anteriores;
2. valorar al cierre actual;
3. calcular NAV_PRE_TRADE;
4. aplicar flujos externos;
5. emitir o retirar unidades, cuando corresponda;
6. cargar la composición EOD canónica;
7. aplicar las transacciones económicas canónicas;
8. calcular cash final;
9. calcular market value final;
10. calcular NAV_POST_TRADE;
11. comprobar continuidad;
12. persistir el snapshot.

No utilizar:

- configured_capital como NAV posterior a inception;
- allocated_notional legacy como valor económico fijo;
- cantidades legacy;
- precio de entrada intradía.

==================================================
6. ORDEN DE CÁLCULO DIARIO
==================================================

Para cada cartera y sesión:

A. ESTADO ANTERIOR

- previous units;
- previous unit value;
- previous cash;
- previous quantities;
- previous NAV.

B. MARK-TO-MARKET

- cierres oficiales actuales;
- market value antes de operaciones;
- pre-flow NAV.

C. FLUJOS EXTERNOS

- aplicar únicamente flujos efectivos de esa sesión;
- calcular units_delta al pre-flow unit value;
- comprobar neutralidad del flujo.

D. TRANSACCIONES INTERNAS

- utilizar solo reconstruction_version = 1;
- aplicar al cierre oficial;
- comprobar NAV continuity;
- comprobar unit continuity;
- comprobar retorno interno cero.

E. ESTADO FINAL

- paper cash;
- paper market value;
- paper NAV;
- units outstanding;
- unit value.

F. RETORNO

daily_return =
unit_value_t / unit_value_t-1 - 1

cumulative_return =
unit_value_t / initial_unit_value - 1

G. RIESGO

- running peak;
- drawdown;
- maximum drawdown.

H. BENCHMARK

- benchmark NAV;
- benchmark units;
- benchmark unit value;
- benchmark daily return;
- benchmark cumulative return;
- benchmark drawdown.

I. ESTADOS

- price coverage;
- historical confidence;
- certification state;
- warnings.

J. PERSISTENCIA

- un único snapshot idempotente.

==================================================
7. ESTADO INICIAL
==================================================

Para cada cartera:

initial_contribution =
configured_capital de su configuración histórica.

No hardcodear 10.000.

initial_unit_value =
100

initial_units =
initial_contribution / 100

Antes de las compras:

paper_cash =
initial_contribution

Después de la primera composición:

paper_market_value =
suma de posiciones compradas al cierre.

paper_NAV =
paper_cash + paper_market_value

Debe cumplirse:

paper_NAV = initial_contribution
paper_unit_value = 100
daily_return = 0
cumulative_return = 0
drawdown = 0
max_drawdown = 0

La aportación y la compra inicial son neutrales al retorno.

==================================================
8. CASH Y REDONDEO
==================================================

Utilizar las precisiones y tolerancias definidas en Gate 2D-1:

- MONETARY_STORAGE_PRECISION;
- QUANTITY_STORAGE_PRECISION;
- ECONOMIC_EPSILON;
- DISPLAY_CASH_EPSILON.

No sustituir todas las cantidades pequeñas por cero durante el cálculo.

Normalizar para presentación o persistencia únicamente según la regla
documentada.

No permitir:

- cash negativo material;
- aportación implícita;
- leverage;
- desaparición de beneficios;
- desaparición de pérdidas;
- corrección silenciosa.

Por snapshot registrar:

- raw cash;
- normalized cash, si el schema lo requiere;
- rounding adjustment;
- warning cuando corresponda.

==================================================
9. CONFIANZA HISTÓRICA POR SESIÓN
==================================================

No marcar automáticamente las cinco series completas como parciales.

Para cada cartera y sesión evaluar las posiciones económicas activas.

Valores:

FULL_HISTORICAL_CONFIDENCE

- todos los objetos económicamente activos tienen confianza completa.

PARTIAL_HISTORICAL_CONFIDENCE

- al menos un objeto activo es UNKNOWN_PRE_LOG o UNKNOWN_UNLINKED;
- la composición y los precios están resueltos;
- el Track puede calcularse, pero requiere advertencia.

BLOCKED

- existe un objeto activo cuya semántica o precio impide el cálculo.

Los 36 objetos aceptados:

- participan en el cálculo;
- no reciben requested_notional inventado;
- conservan MANUAL_REVIEW_REQUIRED;
- no convierten automáticamente sesiones sin exposición suya en parciales.

Persistir o derivar:

- historical_confidence;
- active_partial_confidence_count;
- manual_review_required_count;
- confidence reasons.

Declarar por cartera:

- primera sesión parcial;
- última sesión parcial;
- sesiones full;
- sesiones partial;
- sesiones blocked.

==================================================
10. PRECIO Y ACCIONES CORPORATIVAS
==================================================

PRICE_BASIS =
OFFICIAL_CLOSE_NOMINAL

No usar:

- adjusted total-return;
- precio live;
- apertura;
- precio legacy;
- interpolación;
- previous close de otra sesión.

Dividendos:

- no aumentar cash;
- no reinvertir;
- no modificar units;
- mantener INCOME_NOT_INCLUDED;
- la caída ex-dividendo permanece dentro de price return.

Splits:

- ajustar quantity;
- conservar valor;
- no generar retorno;
- registrar SPLIT_ADJUSTMENT.

Acción no clasificada:

- bloquear snapshot;
- no aplicar fallback.

==================================================
11. BENCHMARK SPY
==================================================

Separar conceptualmente:

benchmark_external_capital

benchmark_spy_shares

benchmark_index_units

benchmark_unit_value

Por cartera:

1. recibir el mismo flujo externo inicial;
2. comprar SPY al cierre oficial de inception;
3. calcular benchmark_spy_shares;
4. fijar benchmark_unit_value inicial = 100;
5. mantener SPY invertido;
6. valorar diariamente.

Las compras y ventas internas de Papel no afectan SPY.

No replicar:

- composiciones;
- rebalanceos;
- memberships;
- reasignaciones;
- paper cash.

Para flujos externos futuros:

- mismo importe;
- misma sesión;
- neutralidad mediante unidades benchmark.

No confundir:

- shares de SPY;
- units del índice benchmark.

Cada snapshot debe poder derivar o contener:

- benchmark NAV;
- benchmark SPY shares;
- benchmark index units;
- benchmark unit value;
- daily return;
- cumulative return;
- drawdown.

No utilizar precios anteriores a inception para retorno.

==================================================
12. SNAPSHOTS
==================================================

Crear un snapshot por:

paper_portfolio_id
+ methodology_id
+ valuation_session
+ economic_reconstruction_version

Verificar si la constraint actual incluye reconstruction version.

Si no la incluye:

- no crear duplicados;
- evaluar migración aditiva mínima;
- utilizar NEXT_AVAILABLE_SCHEMA_VERSION;
- no asumir una versión concreta;
- no ampliar el alcance más allá de Track Papel.

Campos mínimos:

- portfolio;
- methodology;
- economic reconstruction version;
- valuation session;
- paper NAV;
- paper market value;
- paper cash;
- paper units;
- paper unit value;
- daily return;
- cumulative return;
- external flow;
- active economic positions;
- benchmark NAV;
- benchmark SPY shares;
- benchmark index units;
- benchmark unit value;
- benchmark daily return;
- benchmark cumulative return;
- drawdown;
- max drawdown;
- price coverage;
- historical confidence;
- partial confidence count;
- certification state;
- price evidence reference;
- calculated_at;
- content hash.

No usar cero para valores desconocidos.

Utilizar NULL y estado explícito.

==================================================
13. DRAWDOWN
==================================================

running_peak =
máximo unit value hasta la sesión.

drawdown =
unit_value / running_peak - 1

max_drawdown =
mínimo drawdown acumulado.

Convenciones:

- drawdown <= 0;
- max drawdown <= 0;
- primera sesión = 0.

Aplicar la misma convención al benchmark.

No usar cero como fallback si falta información.

==================================================
14. PREVIEW LAB
==================================================

Antes de escribir, reconstruir todas las series en LAB.

Por cartera entregar:

- first session;
- last session;
- expected snapshots;
- initial NAV;
- final NAV;
- initial unit value;
- final unit value;
- daily returns;
- cumulative return;
- final cash;
- final market value;
- max drawdown;
- benchmark final NAV;
- benchmark final return;
- benchmark max drawdown;
- full-confidence sessions;
- partial-confidence sessions;
- blocked sessions;
- warnings.

Comprobar:

- primera sesión = 100;
- aportación neutral;
- compra inicial neutral;
- transacciones internas neutrales;
- último valor marcado a coverage_end_session;
- cat:catalizada-3b deja de aparecer plana si el precio se movió;
- SPY comienza en la inception de cada cartera.

No escribir si:

- existe sesión obligatoria sin precio;
- existe cash negativo material;
- existe NAV discontinuity;
- existe unit discontinuity;
- existe retorno artificial;
- existe doble snapshot;
- benchmark no reconcilia;
- blocked sessions > 0.

==================================================
15. PERSISTENCIA DEL BACKFILL
==================================================

Antes:

- detener escrituras;
- una sola instancia;
- backup;
- SHA-256;
- abrir backup;
- integrity_check;
- foreign_key_check;
- baseline;
- preview LAB verde.

Persistir todos los snapshots dentro de una transacción.

Si falla:

- rollback;
- paper_track_snapshot vuelve a cero;
- estructuras y reconstrucción económica permanecen intactas.

La segunda ejecución:

- devuelve ALREADY_CURRENT;
- no duplica;
- no cambia hashes;
- no altera snapshots válidos.

No modificar un snapshot histórico canónico in-place.

Una corrección futura debe usar:

- nueva reconstruction version;
- supersesión;
- o migración correctiva documentada.

==================================================
16. DAILY-CLOSE
==================================================

Crear:

persistPaperTrackPortfolio

No reutilizar directamente un camino que presuponga Book real.

Responsabilidades:

1. resolver última sesión completamente cerrada;
2. detectar snapshot faltante;
3. resolver composición económica vigente;
4. cargar reconstrucción canónica;
5. obtener cierres oficiales;
6. validar precios;
7. calcular estado diario;
8. calcular benchmark;
9. persistir snapshot;
10. registrar cobertura y confianza.

El orquestador diario debe invocar separadamente:

A. persistencia real existente;
B. persistencia papel.

No crear segundo scheduler.

El fallo papel:

- no impide Track real;
- no impide otras carteras papel;
- queda registrado;
- se reintenta mediante catch-up.

No activar hard cap.

==================================================
17. STARTUP CATCH-UP
==================================================

Al arrancar:

- resolver última sesión cerrada;
- detectar sesiones faltantes por cartera;
- procesarlas cronológicamente;
- usar el snapshot previo;
- no duplicar;
- no depender del navegador;
- no crear posiciones;
- no crear solicitudes;
- no activar políticas.

Si falta precio:

- no usar precio live;
- no usar fallback;
- registrar PRICE_COVERAGE_INCOMPLETE;
- no avanzar la cartera más allá de la sesión faltante;
- reintentar posteriormente.

No crear un snapshot posterior saltando un hueco obligatorio.

==================================================
18. ENDPOINTS READ-ONLY
==================================================

Crear un endpoint canónico principal:

GET /lens/paper-track/:portfolioId

Endpoints adicionales solo si aportan una responsabilidad clara:

GET /lens/paper-track/:portfolioId/summary
GET /lens/paper-track/:portfolioId/episodes
GET /lens/paper-track/:portfolioId/confidence

Los GET deben ser estrictamente read-only.

No pueden:

- generar snapshots;
- descargar precios;
- ejecutar catch-up;
- crear posiciones;
- activar políticas;
- modificar confianza;
- formular tesis.

Payload principal:

- portfolio_id;
- portfolio_kind = PAPER;
- methodology = PRICE_RETURN_TWR;
- reconstruction_version;
- income_policy = INCOME_NOT_INCLUDED;
- benchmark = SPY;
- benchmark_policy;
- first_session;
- last_session;
- as_of;
- series;
- benchmark_series;
- metrics;
- drawdown;
- coverage;
- historical_confidence;
- active_positions;
- warnings;
- certification_state.

Cada punto:

- session;
- NAV;
- unit value;
- daily return;
- cumulative return;
- cash;
- market value;
- active positions;
- drawdown;
- benchmark unit value;
- benchmark cumulative return;
- coverage;
- confidence.

Mantener compatibilidad con consumidores legacy hasta Gate 2E.

==================================================
19. MÉTRICAS
==================================================

Calcular:

- Paper NAV;
- Paper Unit Value;
- retorno diario;
- retorno acumulado;
- retorno desde inception;
- paper cash;
- market value;
- drawdown actual;
- máximo drawdown;
- volatilidad, si puede aplicarse sin reinterpretar metodología;
- benchmark return;
- diferencial simple frente a SPY;
- sesiones valoradas;
- posiciones activas;
- cobertura;
- confianza.

No denominar el diferencial:

alpha

sin metodología estadística.

No mostrar:

- total return;
- dividend-adjusted return;
- métricas no soportadas.

==================================================
20. AISLAMIENTO DE CRECIMIENTO
==================================================

Antes y después comparar:

- payload /track/crecimiento;
- serie;
- fechas;
- métricas;
- ventanas;
- benchmark;
- drawdown;
- valuations;
- holdings;
- movimientos;
- cash;
- NAV;
- Book.

Resultado obligatorio:

GROWTH_TRACK_ECONOMICALLY_IDENTICAL = PASS
REAL_BOOK_UNCHANGED = PASS
REAL_CASH_UNCHANGED = PASS
REAL_NAV_UNCHANGED = PASS
REAL_VALUATIONS_UNCHANGED = PASS

Si los payloads contienen timestamps no deterministas:

- normalizarlos;
- comparar el contenido económico exacto;
- explicar la diferencia.

==================================================
21. PRUEBAS
==================================================

Crear:

verify-paper-track-canonical-reconstruction.mjs
verify-paper-track-engine.mjs
verify-paper-track-daily-series.mjs
verify-paper-track-unitization.mjs
verify-paper-track-benchmark.mjs
verify-paper-track-confidence.mjs
verify-paper-track-drawdown.mjs
verify-paper-track-snapshot-backfill.mjs
verify-paper-track-daily-close.mjs
verify-paper-track-startup-catchup.mjs
verify-paper-track-endpoint.mjs
verify-paper-track-isolation.mjs

Casos mínimos:

A. Solo reconstruction version 1.
B. Primera sesión unit value 100.
C. Aportación neutral.
D. Compra inicial neutral.
E. Sesión sin operación marcada a mercado.
F. Ganancia diaria correcta.
G. Pérdida diaria correcta.
H. Rebalanceo neutral.
I. Pre-fill sin venta en origen.
J. Eventos misma sesión neteados.
K. Cash residual.
L. Cash negativo bloqueado.
M. Retorno acumulado.
N. Drawdown.
O. SPY desde inception.
P. SPY ignora operaciones internas.
Q. Shares SPY distintas de benchmark units.
R. Confianza calculada por sesión.
S. Dividendo excluido.
T. Precio ausente fail-closed.
U. Backfill idempotente.
V. Daily-close aislado.
W. Catch-up cronológico.
X. GET estrictamente read-only.
Y. Crecimiento sin regresión.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- suites v27 y v28;
- suites Gate 2C-2;
- suites Gate 2D-1;
- financial core;
- Growth Track;
- daily-close;
- repository integrity.

==================================================
22. PRODUCCIÓN
==================================================

Después del backfill y despliegue:

- schema válido;
- integrity_check;
- foreign_key_check;
- una sola instancia;
- health;
- env-info;
- snapshots completos;
- endpoint operativo;
- daily-close integrado;
- catch-up idempotente;
- hard cap sin activar;
- legado intacto;
- Crecimiento intacto.

No cambiar todavía la UI.

==================================================
23. CÓDIGO Y COMMIT
==================================================

Añadir:

- motor diario papel;
- propagación del estado económico;
- backfill de snapshots;
- benchmark;
- confianza;
- drawdown;
- adaptador daily-close;
- startup catch-up;
- endpoints;
- read model;
- pruebas;
- documentación.

No añadir:

- UI;
- TrackRecordView;
- hard-cap funcional;
- scratchpad;
- staging;
- backups;
- bases;
- secretos;
- datos productivos innecesarios.

Ejecutar secret scan.

Crear un único commit local:

feat(lens): add daily unitized paper track engine

No crear tag.
No hacer push.

==================================================
24. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = f124193
CANONICAL_HEAD_AFTER = <hash>

SCHEMA_BEFORE = v28
SCHEMA_AFTER = <v28/NEXT_AVAILABLE_SCHEMA_VERSION>

CANONICAL_RECONSTRUCTION_VERSION = 1
SINGLE_CANONICAL_RECONSTRUCTION = PASS/FAIL
LEGACY_TRANSACTIONS_EXCLUDED_FROM_TRACK = PASS/FAIL

ORIGINAL_MANIFEST_END_SESSION = 2026-07-24
CURRENT_COVERAGE_END_SESSION = <fecha>
CURRENT_PRICE_MANIFEST_HASH = <hash>

PAPER_PORTFOLIOS_PROCESSED = <número>/5
PAPER_TRACK_SNAPSHOTS_CREATED = <número>

PAPER_FIRST_SESSION_BY_PORTFOLIO = <tabla>
PAPER_LAST_SESSION_BY_PORTFOLIO = <tabla>
PAPER_SNAPSHOT_COUNT_BY_PORTFOLIO = <tabla>

PAPER_INITIAL_UNIT_VALUE = 100
PAPER_FINAL_UNIT_VALUE_BY_PORTFOLIO = <tabla>
PAPER_CUMULATIVE_RETURN_BY_PORTFOLIO = <tabla>
PAPER_FINAL_NAV_BY_PORTFOLIO = <tabla>
PAPER_FINAL_CASH_BY_PORTFOLIO = <tabla>
PAPER_MAX_DRAWDOWN_BY_PORTFOLIO = <tabla>

SPY_SERIES_COMPLETE = <número>/5
SPY_FINAL_RETURN_BY_PORTFOLIO = <tabla>
SPY_MAX_DRAWDOWN_BY_PORTFOLIO = <tabla>
SPY_SHARES_UNITS_SEPARATED = PASS/FAIL

FULL_CONFIDENCE_SESSIONS = <número>
PARTIAL_CONFIDENCE_SESSIONS = <número>
BLOCKED_SESSIONS = <número>
MANUAL_REVIEW_REQUIRED_OBJECTS = 36

PRICE_COVERAGE_COMPLETE = PASS/FAIL
PRE_FILL_REASSIGNMENT_ZERO_ORIGIN_EXPOSURE = PASS/FAIL
SAME_SESSION_NETTING_PRESERVED = PASS/FAIL
NO_ARTIFICIAL_TURNOVER = PASS/FAIL

SNAPSHOT_BACKFILL_TRANSACTIONAL = PASS/FAIL
SNAPSHOT_BACKFILL_IDEMPOTENT = PASS/FAIL

DAILY_CLOSE_PAPER_ADAPTER = PASS/FAIL
STARTUP_CATCHUP_PAPER = PASS/FAIL
PAPER_TRACK_ENDPOINT = <ruta>
PAPER_TRACK_ENDPOINT_GREEN = PASS/FAIL
GET_ENDPOINTS_READ_ONLY = PASS/FAIL

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

NO_POLICY_ACTIVATED = PASS/FAIL
GROWTH_TRACK_ECONOMICALLY_IDENTICAL = PASS/FAIL
REAL_BOOK_UNCHANGED = PASS/FAIL
REAL_CASH_UNCHANGED = PASS/FAIL
REAL_NAV_UNCHANGED = PASS/FAIL
REAL_VALUATIONS_UNCHANGED = PASS/FAIL
HOLDINGS_LEGACY_UNCHANGED = PASS/FAIL
SNAPSHOTS_LEGACY_UNCHANGED = PASS/FAIL
THESES_UNCHANGED = PASS/FAIL
CATALYSTS_UNCHANGED = PASS/FAIL
HUNTING_FIELD_UNCHANGED = PASS/FAIL
NO_REAL_ECONOMIC_WRITES = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

LOCAL_COMMIT = <hash>

EXACT_BLOCKERS_BEFORE_GATE_2E:
1. <bloqueo>
2. <bloqueo>
3. <bloqueo>

Detenerse después del Gate 2D-2.