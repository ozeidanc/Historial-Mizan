MIZAN · LA LENTE
GATE 2C-1 · COBERTURA DE PRECIOS POINT-IN-TIME
STAGING AISLADO + PREVIEW COMPLETO · SIN BACKFILL

NATURALEZA

Esta ejecución autoriza:

1. inventariar todas las necesidades históricas de precios;
2. deduplicar solicitudes por símbolo, sesión y base;
3. obtener precios mediante el proveedor canónico existente;
4. almacenar los resultados únicamente en staging/LAB aislado;
5. validar sesiones, símbolos, cierres y cobertura;
6. generar un preview completo del futuro backfill;
7. crear tooling y pruebas reutilizables;
8. crear, si todo queda verde, un único commit local de código.

No autoriza:

- escribir precios en tablas productivas;
- escribir en la caché productiva;
- crear global positions en Producción;
- crear membership episodes en Producción;
- crear allocation requests en Producción;
- crear configuraciones reales;
- crear composiciones;
- crear flows;
- crear transactions;
- crear track snapshots;
- ejecutar backfill;
- activar MANUAL_NOTIONAL_HARD_CAP;
- establecer effective_from;
- cambiar daily-close;
- cambiar endpoints funcionales;
- cambiar UI;
- modificar Track de Crecimiento;
- modificar holdings;
- modificar tesis;
- modificar catalizadores;
- modificar Campo de Caza;
- hacer deploy;
- crear tag;
- hacer push;
- realizar operaciones remotas Git.

Los accesos de red quedan autorizados exclusivamente para obtener datos de
mercado mediante la infraestructura y los proveedores ya permitidos por
Mizan.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = 032a191
SCHEMA_PRODUCTION esperado = v27
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Gate 2B confirmado:

- schema v27;
- metodología PAPER_PRICE_RETURN_TWR_V1 sellada;
- política histórica legacy disponible solo para replay;
- política prospectiva MANUAL_NOTIONAL_HARD_CAP pendiente de activación;
- 11 tablas v27;
- 9 tablas operativas vacías;
- cero backfill;
- cero precios obtenidos;
- cero políticas activas.

Inventario histórico confirmado:

GLOBAL_PAPER_POSITIONS = 82
CURRENT_OPEN_GLOBAL_POSITIONS = 82

PORTFOLIO_MEMBERSHIP_EPISODES_PROPOSED = 121
CURRENT_OPEN_MEMBERSHIP_EPISODES = 82
CLOSED_MEMBERSHIP_EPISODES = 39
INTER_PORTFOLIO_REASSIGNMENTS = 39
ECONOMICALLY_CLOSED_GLOBAL_POSITIONS = 0

REQUESTED_NOTIONAL_KNOWN = 46
REQUESTED_NOTIONAL_UNKNOWN = 36
REQUESTED_UNKNOWN_PRE_LOG = 26
REQUESTED_UNKNOWN_UNLINKED = 10

Metodología:

HISTORICAL_TRACK_POLICY =
AS_OPERATED

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

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

Antes de obtener precios:

- confirmar HEAD 032a191;
- confirmar env-info;
- confirmar schema v27;
- confirmar una única raíz Git;
- inventariar untracked existentes;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- confirmar una sola instancia;
- confirmar health;
- comprobar que las nueve tablas operativas v27 siguen vacías;
- capturar contadores y hashes productivos;
- confirmar que la política prospectiva sigue sin effective_from.

Capturar al menos:

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

No continuar si:

- el schema no es v27;
- HEAD o env-info no coinciden;
- integrity_check falla;
- existen violaciones de foreign keys;
- alguna tabla operativa v27 contiene filas no autorizadas;
- la política prospectiva aparece activa;
- el staging no está aislado de Producción.

Declarar:

PRODUCTION_READ_ONLY = PASS/FAIL
PRICE_STAGING_ISOLATED = PASS/FAIL

==================================================
2. FECHA FINAL DE COBERTURA
==================================================

Resolver mediante el calendario NYSE existente:

coverage_end_session =
última sesión bursátil America/New_York completamente cerrada en el momento
de ejecución.

No utilizar:

- fecha UTC actual directamente;
- fecha de Dubái;
- fecha natural del servidor;
- una sesión todavía abierta;
- datos intradía como cierre certificado.

La ejecución se realiza el domingo 26 de julio de 2026.

El resolver debería identificar la última sesión completamente cerrada
anterior, pero debe validarlo con el calendario canónico y no codificar una
fecha fija.

Persistir en el preview:

- execution_time_utc;
- execution_time_dubai;
- execution_time_new_york;
- coverage_end_session;
- calendar_source.

==================================================
3. UNIVERSO COMPLETO DE NECESIDADES DE PRECIOS
==================================================

No limitar el inventario a:

- 80 anchors pendientes;
- 39 sesiones de reasignación.

Construir cuatro conjuntos.

A. ANCHOR REQUIREMENTS

Para cada global position o membership inicial:

- ticker;
- incorporated_at_utc;
- market state;
- anchor_session;
- anchor price requerido;
- reason = POSITION_ANCHOR.

B. REASSIGNMENT REQUIREMENTS

Para cada una de las 39 reasignaciones:

- reassignment_group_id;
- ticker;
- origin portfolio;
- destination portfolio;
- effective timestamp;
- effective session;
- cierre requerido;
- reason = MEMBERSHIP_REASSIGNMENT.

La reasignación debe generar una única necesidad económica de precio por:

ticker + effective_session + price_basis

aunque sea utilizada por origen y destino.

C. REBALANCE REQUIREMENTS

Para cada versión histórica legacy reconstruible:

- cartera;
- effective session;
- composición anterior;
- composición posterior;
- tickers que requieren compra;
- tickers que requieren venta;
- cierres requeridos;
- reason = LEGACY_REBALANCE.

No asumir que el precio del ticker incorporado es el único precio requerido.

Un rebalanceo capital/N puede exigir precios para todas las posiciones cuya
cantidad cambia.

D. DAILY VALUATION REQUIREMENTS

Para cada membership episode propuesto:

- ticker;
- first effective valuation session;
- last effective valuation session;
- todas las sesiones bursátiles comprendidas;
- cierres diarios necesarios.

Para memberships actualmente abiertos:

last effective valuation session =
coverage_end_session.

Para memberships cerrados por reasignación:

last effective valuation session =
effective_session de salida según el contrato temporal.

E. BENCHMARK REQUIREMENTS

Para SPY:

- sesión de aportación inicial de cada cartera;
- todas las sesiones diarias de valoración;
- futuras sesiones de flujo externo, si existieran;
- cobertura completa hasta coverage_end_session.

Entregar antes del fetch:

RAW_PRICE_REQUIREMENTS
UNIQUE_PRICE_REQUIREMENTS
DUPLICATE_REQUIREMENTS_REMOVED
UNIQUE_TICKERS
UNIQUE_SESSIONS
EARLIEST_REQUIRED_SESSION
LATEST_REQUIRED_SESSION

==================================================
4. DEDUPLICACIÓN
==================================================

Deduplicar por:

- canonical_ticker;
- valuation_session;
- price_basis;
- provider.

No deduplicar únicamente por ticker.

Una misma cotización puede satisfacer múltiples usos:

- anchor;
- reasignación;
- rebalanceo;
- valoración diaria;
- benchmark.

Conservar una tabla de referencias:

price_requirement_usage

con:

- requirement key;
- usage type;
- global position;
- membership;
- reassignment;
- composition version;
- portfolio;
- benchmark;
- reason.

No crear esta tabla en Producción durante Gate 2C-1.

Puede existir solo en staging o como artefacto de preview.

==================================================
5. PROVEEDOR Y CONTRATO DEL PRECIO
==================================================

Reutilizar exactamente la infraestructura de precios de Crecimiento.

Determinar y reportar:

- provider;
- endpoint o cliente existente;
- símbolo solicitado;
- símbolo devuelto;
- exchange;
- currency;
- timezone;
- session date;
- raw close;
- normalized close;
- split handling;
- response timestamp;
- cache status;
- source status.

Contrato:

PRICE_BASIS =
OFFICIAL_CLOSE_NOMINAL

En este contexto “official close” significa:

- cierre EOD canónico suministrado por el proveedor autorizado de Mizan;
- alineado con la sesión del calendario NYSE;
- usando la misma base de precio que Crecimiento.

No presentarlo como:

- ejecución real;
- precio intradía;
- total-return price;
- dividend-adjusted return.

No usar fallback silencioso a:

- precio live;
- apertura;
- previous close de otra sesión;
- precio de La Lente;
- precio almacenado en holdings;
- precio de entrada original;
- SPY actual;
- interpolación.

==================================================
6. STAGING DE PRECIOS
==================================================

Guardar resultados únicamente en:

- memoria;
- base LAB;
- o scratchpad/gate2c1/price-staging.*

No escribir en:

- tablas v27 productivas;
- valuations productivas;
- caché productiva;
- holdings;
- snapshots;
- tesis;
- cat_composicion_log.

Cada registro de staging debe incluir:

- requested_ticker;
- canonical_ticker;
- provider_symbol;
- valuation_session;
- price_basis;
- close;
- currency;
- provider;
- fetched_at;
- source_payload_hash;
- coverage_status;
- validation_status;
- warning;
- usage_count.

No incluir secretos ni tokens.

No guardar respuestas completas del proveedor si contienen metadatos
innecesarios o información sensible.

==================================================
7. VALIDACIÓN DE SESIONES
==================================================

Para cada precio:

- comprobar que valuation_session es sesión bursátil válida;
- comprobar que el proveedor devuelve esa misma sesión;
- comprobar que no se desplazó por UTC;
- comprobar timezone America/New_York;
- comprobar que no es fin de semana;
- comprobar que no es festivo;
- comprobar que el cierre era conocido al momento de usarlo;
- comprobar que no existe look-ahead.

Estados:

- VALIDATED;
- SESSION_MISMATCH;
- NON_TRADING_SESSION;
- FUTURE_OR_OPEN_SESSION;
- PROVIDER_DATE_SHIFT;
- MISSING.

No mover automáticamente una solicitud a otra fecha si falta el precio.

Una corrección de sesión debe proceder del calendario y quedar registrada con
motivo explícito.

==================================================
8. SÍMBOLOS Y CORPORATE ACTIONS
==================================================

Auditar:

- cambios de ticker;
- fusiones;
- adquisiciones;
- delistings;
- splits;
- reverse splits;
- símbolos con sufijos;
- distintas bolsas;
- ADR;
- moneda distinta de USD;
- duplicados de símbolo.

Para cada caso problemático entregar:

- ticker histórico;
- ticker actual;
- sesión afectada;
- mapping propuesto;
- evidencia del mapping;
- precio antes;
- precio después;
- tratamiento requerido;
- confianza.

No sustituir automáticamente un ticker por otro sin evidencia.

SPLITS

Usar la misma metodología de Crecimiento.

Comprobar que:

- el cierre nominal y la cantidad quedan en base compatible;
- el split no genera retorno artificial;
- las cantidades anteriores y posteriores pueden reconstruirse.

DIVIDENDOS

Detectarlos únicamente para advertencia de metodología.

Como:

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

los dividendos no se incorporan al Track v1.

Persistir en el preview:

- episodios afectados;
- sesiones;
- warning INCOME_NOT_INCLUDED.

No convertir el precio a adjusted total-return.

==================================================
9. REASIGNACIONES
==================================================

Para cada una de las 39 reasignaciones:

- usar reassignment_group_id;
- resolver un único effective timestamp;
- resolver una única effective_session;
- obtener un único cierre canónico por ticker/sesión;
- reutilizarlo en origen y destino;
- marcar ambas patas como parte del mismo evento atómico.

Preview de origen:

- membership cerrado;
- composición antes/después;
- allocated notional antes/después;
- cantidad antes/después;
- venta o rebalanceo interno requerido;
- paper cash resultante;
- cero flujo externo.

Preview de destino:

- membership abierto;
- composición antes/después;
- allocated notional antes/después;
- cantidad antes/después;
- compra o rebalanceo interno requerido;
- paper cash resultante;
- cero flujo externo.

No crear todavía:

- membership;
- transaction;
- composition version;
- flow.

Si falta cualquier precio requerido por una reasignación:

REASSIGNMENT_PRICE_COVERAGE =
INCOMPLETE

La reasignación completa no puede considerarse certificable parcialmente.

==================================================
10. REQUESTED NOTIONAL DESCONOCIDO
==================================================

Mantener:

KNOWN = 46
UNKNOWN_PRE_LOG = 26
UNKNOWN_UNLINKED = 10

Para UNKNOWN_PRE_LOG:

requested_notional = NULL
reconstruction_confidence = INFERRED
certification_state = MANUAL_REVIEW_REQUIRED

Para UNKNOWN_UNLINKED:

requested_notional = NULL
reconstruction_confidence =
DETERMINISTIC o INFERRED según evidencia disponible

certification_state =
MANUAL_REVIEW_REQUIRED salvo demostración adicional explícita.

La obtención de precios no cambia estos estados.

No usar cobertura completa de precios como prueba del requested_notional.

Puede calcularse en el preview:

candidate_allocated_notional

según la política legacy y la composición reconstruida.

Pero debe permanecer separado de la solicitud de Omar.

==================================================
11. PREVIEW DEL BACKFILL
==================================================

Generar un preview completo sin escribirlo en Producción.

A. GLOBAL POSITIONS

Para cada una:

- identity;
- ticker;
- thesis;
- incorporated_at;
- anchor;
- cobertura;
- requested status;
- candidate certification state.

B. MEMBERSHIP EPISODES

Para los 121 propuestos:

- origin/destination;
- entered_at;
- exited_at;
- entered_session;
- exited_session;
- anchor;
- reassignment group;
- coverage;
- confidence.

C. COMPOSITION VERSIONS

Por cartera y evento:

- policy;
- effective_at;
- effective_session;
- members before;
- members after;
- configured capital;
- N;
- allocated notionals;
- quantities;
- turnover;
- required prices;
- coverage.

D. TRANSACTIONS

Proponer, sin crear:

- PURCHASE;
- SALE;
- LEGACY_REBALANCE_BUY;
- LEGACY_REBALANCE_SELL;
- MEMBERSHIP_TRANSFER_OUT;
- MEMBERSHIP_TRANSFER_IN.

Confirmar si TRANSFER_OUT/IN deben ser tipos propios o razones de
PURCHASE/SALE.

No duplicar el efecto económico.

E. INITIAL FLOWS

Proponer, sin crear:

- aportación inicial por cartera;
- effective session;
- configured capital;
- unidades iniciales;
- benchmark SPY.

No asumir que las cinco aportaciones tienen la misma sesión.

F. DAILY SERIES FEASIBILITY

Por cartera:

- sesiones requeridas;
- sesiones cubiertas;
- porcentaje;
- primer día;
- último día;
- gaps;
- episodios incompletos;
- benchmark coverage;
- factibilidad de reconstrucción diaria.

==================================================
12. CLASIFICACIÓN DE COBERTURA
==================================================

Clasificar cada objeto:

PRICE_COVERAGE_COMPLETE

Todos los precios requeridos están validados.

PRICE_COVERAGE_PARTIAL

Faltan precios, pero los disponibles son válidos.

PRICE_COVERAGE_BLOCKED

Existe conflicto de sesión, ticker, corporate action o proveedor.

MANUAL_REVIEW_REQUIRED

El precio puede existir, pero la semántica histórica no es certificable
automáticamente.

No equiparar:

PRICE_COVERAGE_COMPLETE
con
HISTORY_CERTIFIED.

Son dimensiones diferentes.

==================================================
13. CRITERIOS DE CANDIDATURA PARA BACKFILL
==================================================

AUTO_BACKFILL_CANDIDATE

Solo si:

- lineage resuelto;
- sesiones resueltas;
- precios completos;
- símbolo validado;
- policy histórica demostrable;
- requested status compatible;
- allocated notional reconstruible;
- corporate actions resueltas;
- cero conflicto;
- certification state compatible.

MANUAL_REVIEW_REQUIRED

Incluye, como mínimo:

- 26 pre-log INFERRED;
- requested desconocido sin evidencia suficiente;
- mapping de símbolo no demostrado;
- corporate action no resuelta;
- inconsistencia entre log, holding y snapshot.

PRICE_COVERAGE_INCOMPLETE

Falta al menos un precio obligatorio.

BLOCKED

Existe una contradicción que impide proponer el backfill.

Gate 2C-1 no ejecuta ninguna de estas categorías.

==================================================
14. PRUEBAS
==================================================

Crear:

verify-paper-price-requirements.mjs
verify-paper-price-staging.mjs
verify-paper-price-sessions.mjs
verify-paper-price-coverage-preview.mjs
verify-paper-reassignment-price-preview.mjs
verify-paper-daily-series-feasibility.mjs

Casos mínimos:

A. DEDUPLICACIÓN

Un precio usado por anchor, rebalanceo y daily valuation se obtiene una vez y
conserva tres usages.

B. FECHA UTC DISTINTA DE NY

Usar sesión America/New_York.

C. FIN DE SEMANA

No solicitar cierre para sábado o domingo.

D. FESTIVO

No crear sesión artificial.

E. SESIÓN ABIERTA

No certificar cierre.

F. PRECIO AUSENTE

No fallback silencioso.

G. REASIGNACIÓN

Mismo precio para origen y destino; evento atómico.

H. TICKER CHANGE

No mapear sin evidencia.

I. SPLIT

No generar retorno artificial.

J. DIVIDENDO

Advertir INCOME_NOT_INCLUDED; no usar total-return adjusted close.

K. REQUESTED DESCONOCIDO

Precio completo no cambia requested_notional=NULL.

L. DAILY COVERAGE

Validar todas las sesiones del intervalo, no solo el anchor.

M. AISLAMIENTO

Cero escrituras en Producción y caché productiva.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

==================================================
15. ARTEFACTOS
==================================================

Crear en:

scratchpad/gate2c1/

Como mínimo:

- price-requirements.json;
- price-usage-map.json;
- price-staging.json o base staging;
- coverage-summary.json;
- anchor-preview.json;
- reassignment-preview.json;
- composition-preview.json;
- daily-series-feasibility.json;
- manual-review.json;
- blocked-cases.json.

No incluir:

- secretos;
- tokens;
- .env;
- base productiva;
- backups completos;
- payloads innecesarios;
- datos personales.

No añadir estos artefactos al commit.

==================================================
16. CÓDIGO Y COMMIT LOCAL
==================================================

Puede añadirse al repositorio:

- tooling reutilizable de requerimientos de precios;
- validadores de sesiones;
- staging aislado;
- generadores de preview;
- suites de pruebas;
- documentación técnica.

No integrar todavía el tooling en:

- daily-close;
- startup;
- endpoints;
- UI.

Ejecutar secret scan.

Si todas las suites están verdes, crear un único commit local:

feat(lens): add paper track price coverage preview tooling

No crear tag.
No hacer push.

Si el tooling no está suficientemente estable:

- no crear commit;
- entregar artefactos y bloqueos;
- mantener los archivos untracked.

==================================================
17. VERIFICACIÓN FINAL DE PRODUCCIÓN
==================================================

Después del fetch y preview:

- schema sigue en v27;
- integrity_check;
- foreign_key_check;
- una sola instancia;
- health;
- env-info;
- política hard cap sigue sin effective_from;
- tablas operativas v27 siguen vacías;
- contadores históricos idénticos;
- hashes de Crecimiento idénticos;
- cash real idéntico;
- NAV real idéntico;
- Track de Crecimiento idéntico.

Contadores obligatorios:

paper_portfolio_track_config = 0
paper_global_position = 0
paper_portfolio_membership_episode = 0
paper_allocation_request = 0
paper_composition_version = 0
paper_composition_member = 0
paper_portfolio_flow = 0
paper_portfolio_transaction = 0
paper_track_snapshot = 0

==================================================
18. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = 032a191
CANONICAL_HEAD_AFTER = <hash/UNCHANGED>

SCHEMA_BEFORE = v27
SCHEMA_AFTER = v27

COVERAGE_END_SESSION = <fecha>
PRICE_PROVIDER = <proveedor>
PRICE_BASIS = OFFICIAL_CLOSE_NOMINAL

RAW_PRICE_REQUIREMENTS = <número>
UNIQUE_PRICE_REQUIREMENTS = <número>
DUPLICATE_REQUIREMENTS_REMOVED = <número>
UNIQUE_TICKERS = <número>
UNIQUE_SESSIONS = <número>

ANCHOR_REQUIREMENTS = <número>
ANCHOR_PRICES_VALIDATED = <número>
ANCHOR_PRICES_MISSING = <número>
ANCHOR_PRICES_BLOCKED = <número>

REASSIGNMENTS = 39
REASSIGNMENTS_PRICE_COMPLETE = <número>
REASSIGNMENTS_PRICE_INCOMPLETE = <número>
REASSIGNMENTS_BLOCKED = <número>

DAILY_PRICE_REQUIREMENTS = <número>
DAILY_PRICES_VALIDATED = <número>
DAILY_PRICES_MISSING = <número>
DAILY_SERIES_COVERAGE = <porcentaje>

SPY_REQUIRED_SESSIONS = <número>
SPY_VALIDATED_SESSIONS = <número>
SPY_COVERAGE = <porcentaje>

SYMBOL_MAPPING_CASES = <número>
SPLIT_CASES = <número>
DIVIDEND_WARNING_CASES = <número>
PRICE_GAP_CASES = <número>

GLOBAL_POSITIONS_PREVIEWED = <número>
MEMBERSHIP_EPISODES_PREVIEWED = <número>
COMPOSITION_VERSIONS_PREVIEWED = <número>
TRANSACTIONS_PREVIEWED = <número>

AUTO_BACKFILL_CANDIDATES = <número>
MANUAL_REVIEW_REQUIRED = <número>
PRICE_COVERAGE_INCOMPLETE = <número>
BLOCKED_CASES = <número>

REQUESTED_NOTIONAL_KNOWN = 46
REQUESTED_NOTIONAL_UNKNOWN_PRE_LOG = 26
REQUESTED_NOTIONAL_UNKNOWN_UNLINKED = 10

PRODUCTION_PRICE_CACHE_WRITES = 0
PRODUCTION_OPERATIONAL_TABLE_WRITES = 0
NO_BACKFILL_EXECUTED = PASS/FAIL
NO_POLICY_ACTIVATED = PASS/FAIL
NO_PAPER_HISTORY_MUTATED = PASS/FAIL
NO_GROWTH_TRACK_CHANGED = PASS/FAIL
NO_REAL_DATA_MUTATED = PASS/FAIL
NO_ECONOMIC_WRITES = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

LOCAL_COMMIT = <hash/NONE>

EXACT_BLOCKERS_BEFORE_GATE_2C2:
1. <bloqueo>
2. <bloqueo>
3. <bloqueo>

Detenerse después de Gate 2C-1.