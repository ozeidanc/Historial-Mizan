MIZAN · LA LENTE
GATE 2D · MOTOR DIARIO DEL TRACK RECORD PAPEL
PRICE RETURN TWR · SNAPSHOTS EOD · BENCHMARK SPY · ENDPOINTS

BASELINE

CANONICAL_HEAD esperado = 66059a9
schema = v27
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

GATE 2C-2 ACEPTADO

Inventario estructural confirmado:

- 5 configuraciones históricas;
- 5 aportaciones iniciales;
- 82 posiciones globales abiertas;
- 121 membership episodes;
- 82 memberships abiertos;
- 39 memberships cerrados por reasignación previa al fill;
- 82 allocation requests;
- 19 composiciones económicas EOD;
- 249 composition members;
- 249 transacciones internas netas;
- 863 puntos de evidencia de precio;
- cobertura de precios = 100 %;
- paper_track_snapshot = 0.

Confianza histórica:

REQUESTED_NOTIONAL_KNOWN = 46

REQUESTED_NOTIONAL_UNKNOWN_PRE_LOG = 26
REQUESTED_NOTIONAL_UNKNOWN_UNLINKED = 10

Los 36 casos desconocidos quedan aprobados para participar en la
reconstrucción AS_OPERATED con:

requested_notional = NULL

certification_state =
MANUAL_REVIEW_REQUIRED

historical_confidence =
PARTIAL_HISTORICAL_CONFIDENCE

No convertirlos en CERTIFIED.

Decisión autorizada:

GENERATE_PAPER_TRACK_SNAPSHOTS = YES

Decisión aplazada:

MANUAL_NOTIONAL_HARD_CAP.status =
APPROVED_PENDING_ACTIVATION

MANUAL_NOTIONAL_HARD_CAP.effective_from =
NULL

No activar la política prospectiva en Gate 2D.

==================================================
0. MISIÓN
==================================================

Construir el motor diario certificado del Track Papel:

1. reconstruir PAPER NAV por cartera y sesión;
2. calcular PRICE_RETURN_TWR unitizado;
3. calcular retorno diario y acumulado;
4. calcular drawdown;
5. construir benchmark SPY con los mismos flujos externos;
6. persistir paper_track_snapshot;
7. extender daily-close mediante un adaptador papel explícito;
8. implementar startup catch-up;
9. exponer endpoints normalizados de lectura;
10. validar que el Track de Crecimiento no cambia.

Este Gate genera los datos correctos para las gráficas.

No implementa todavía la paridad visual final.

La sustitución de papelChartHTML y el componente TrackRecordView compartido
corresponden a Gate 2E.

==================================================
1. RESTRICCIONES
==================================================

No modificar:

- holdings legacy;
- snapshots legacy;
- cat_composicion_log;
- tesis;
- catalizadores;
- Campo de Caza;
- Book real;
- cash real;
- NAV real;
- movimientos reales;
- decisiones reales;
- resizing;
- Track histórico de Crecimiento;
- metodología de Crecimiento;
- políticas de Crecimiento;
- selector C0;
- Defensiva;
- Equilibrada.

No activar:

MANUAL_NOTIONAL_HARD_CAP

No fijar:

effective_from

No crear:

- solicitudes prospectivas;
- nuevas posiciones papel;
- nuevos memberships;
- nuevas composiciones históricas;
- aportaciones externas no demostradas;
- operaciones Wio;
- operaciones reales.

No hacer push.
No crear tag.
No realizar operaciones Git remotas.

==================================================
2. PRECONDICIONES
==================================================

Antes de modificar:

- confirmar HEAD 66059a9;
- confirmar env-info;
- confirmar schema v27;
- confirmar una sola raíz Git;
- inventariar archivos untracked;
- confirmar una sola instancia;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- confirmar health;
- validar el manifiesto de precios;
- validar su hash;
- validar los contadores del backfill;
- validar las 19 composiciones EOD;
- validar las 249 transacciones;
- confirmar paper_track_snapshot = 0;
- confirmar hard cap sin effective_from.

Capturar baseline completo de Crecimiento:

- payload de /track/crecimiento;
- serie;
- fechas;
- métricas;
- ventanas;
- benchmark;
- drawdown;
- cobertura;
- valuations;
- holdings;
- movimientos;
- cash;
- NAV;
- hashes.

Capturar baseline legacy de Papel:

- endpoints actuales;
- payload actual;
- gráfica actual;
- contadores;
- serie actual incompleta.

No continuar si:

- el manifiesto de precios no coincide con su hash;
- algún contador estructural difiere;
- existe una acción corporativa sin resolver;
- integrity_check falla;
- foreign_key_check devuelve violaciones;
- hard cap está activo.

==================================================
3. FUENTES CANÓNICAS DEL MOTOR
==================================================

El motor papel debe leer exclusivamente:

- paper_portfolio_track_config;
- paper_portfolio_flow;
- paper_composition_version;
- paper_composition_member;
- paper_portfolio_transaction;
- paper_global_position;
- paper_portfolio_membership_episode;
- metodología sellada;
- políticas históricas;
- evidencia validada de precios.

No usar como fuente económica directa para reconstrucción:

- holdings actuales;
- snapshots legacy;
- composición final aplicada hacia atrás;
- precio_entrada legacy;
- precio live;
- cat_composicion_log sin pasar por la composición EOD reconstruida.

cat_composicion_log sigue siendo:

INTRADAY_AUDIT_SOURCE

Las 19 filas de composición deben declararse:

EOD_COMPOSITION_VERSIONS

No denominarlas auditoría intradía completa.

El Track diario debe basarse en la última composición económica EOD de cada
cartera y sesión.

==================================================
4. MEMBERSHIPS SIN EXPOSICIÓN ECONÓMICA
==================================================

Los 39 memberships cerrados por reasignación ocurrieron antes del fill.

Por tanto:

- aportan trazabilidad de pertenencia;
- no implican exposición económica en origen;
- no generan market value en origen;
- no generan P&L en origen;
- no generan venta en origen;
- no generan turnover en origen;
- no entran en el Track de origen antes del fill.

La exposición económica debe determinarse mediante:

- composition member EOD;
- certified quantity;
- effective session;
- transacción económica neta.

No inferir exposición solo porque exista un membership episode.

Crear una prueba específica:

PRE_FILL_REASSIGNMENT_HAS_ZERO_ORIGIN_EXPOSURE

==================================================
5. CALENDARIO DE VALORACIÓN
==================================================

Para cada cartera:

first_valuation_session =
sesión de la aportación inicial y primera composición económica.

last_historical_session =
2026-07-24 para el backfill inicial.

Posteriormente:

last_valuation_session =
última sesión NYSE completamente cerrada.

Utilizar el calendario corregido con:

- sesiones regulares;
- early close;
- festivos;
- fines de semana;
- DST;
- America/New_York.

No generar:

- puntos intradía certificados;
- puntos en días no operativos;
- puntos para una sesión todavía abierta.

La curva certificada finaliza siempre en la última sesión completamente
cerrada.

==================================================
6. ORDEN ECONÓMICO DIARIO
==================================================

Para cada cartera y sesión:

1. cargar el snapshot de la sesión anterior;
2. cargar cantidades EOD heredadas;
3. obtener cierres oficiales de la sesión actual;
4. valorar la composición anterior al cierre actual;
5. reconocer el movimiento de precio;
6. reconocer income soportado;
7. aplicar flujos externos efectivos de la sesión;
8. emitir o retirar unidades por flujos externos;
9. aplicar la composición EOD final;
10. aplicar transacciones netas al cierre;
11. calcular paper cash;
12. calcular market value final;
13. calcular NAV final;
14. calcular unit value;
15. calcular retorno diario;
16. calcular retorno acumulado;
17. calcular drawdown;
18. calcular benchmark;
19. calcular cobertura y confianza;
20. persistir un snapshot idempotente.

Las transacciones al cierre:

- no crean retorno por sí mismas;
- no cambian unit value;
- transforman cash en market value o viceversa;
- usan el mismo cierre que la composición EOD.

==================================================
7. ESTADO INICIAL
==================================================

Por cartera:

initial_contribution =
configured_capital histórico.

initial_unit_value =
100

initial_units =
initial_contribution / 100

Antes de compras:

paper_cash =
initial_contribution

Después de la primera composición EOD:

paper_cash =
initial contribution - compras netas

paper_market_value =
valor de posiciones adquiridas

paper_NAV =
paper_cash + paper_market_value

paper_unit_value debe permanecer:

100

La aportación inicial y las compras iniciales no generan rentabilidad.

El primer snapshot puede registrarse con:

daily_return = 0
cumulative_return = 0
drawdown = 0

==================================================
8. PAPER NAV Y UNITIZACIÓN
==================================================

Identidad:

paper_NAV_t =
paper_cash_t
+ paper_market_value_t

units_before_flow =
units de sesión anterior

Cuando no existen flujos externos:

paper_unit_value_t =
paper_NAV_t / units_outstanding_t

Cuando existe flujo externo al cierre:

pre_flow_unit_value =
pre_flow_NAV / units_before_flow

units_delta =
external_flow / pre_flow_unit_value

units_after =
units_before_flow + units_delta

El flujo externo no cambia unit value.

Retorno diario:

daily_return_t =
paper_unit_value_t / paper_unit_value_t-1 - 1

Retorno acumulado:

cumulative_return_t =
paper_unit_value_t / 100 - 1

No denominarlo total return.

Etiqueta:

PRICE_RETURN_TWR

Advertencia:

INCOME_NOT_INCLUDED

==================================================
9. PAPER CASH
==================================================

El cash inicial procede del flujo externo inicial.

Compras internas:

- reducen paper cash.

Ventas internas:

- aumentan paper cash.

Rebalanceos:

- aplican deltas netos;
- no crean flujos externos.

Reasignaciones previas al fill:

- no crean venta económica en origen;
- la compra ocurre únicamente en el destino.

No permitir:

- cash negativo fuera de tolerancia;
- aportación implícita;
- leverage;
- corrección silenciosa.

Tolerancia:

- utilizar la misma tolerancia monetaria documentada en Gate 2C-2;
- normalizar únicamente residuos de redondeo dentro de esa tolerancia;
- registrar rounding adjustment;
- no ocultar diferencias económicas.

==================================================
10. PRECIOS
==================================================

Para el backfill inicial usar exactamente los precios validados por:

PRICE_MANIFEST_HASH =
3c5bf06f68db1021a491dc511470eb1a29b0c8be81bf53165ee3c1ab21cc943a

Para sesiones futuras:

- utilizar el proveedor canónico;
- OFFICIAL_CLOSE_NOMINAL;
- calendario NYSE;
- mismo contrato de validación;
- sin fallback silencioso.

No usar:

- adjusted close total-return;
- precio live;
- apertura;
- precio de entrada legacy;
- caché indicativa;
- interpolación.

Cada snapshot debe conservar trazabilidad suficiente:

- price manifest/reference;
- price basis;
- provider;
- coverage status;
- calculated_at;
- content hash.

No es necesario duplicar todos los precios dentro del snapshot, pero debe
poder reconstruirse qué evidencia se utilizó.

==================================================
11. DIVIDENDOS Y ACCIONES CORPORATIVAS
==================================================

Metodología actual:

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

Por tanto:

- no añadir dividendos a paper cash;
- no reinvertir dividendos;
- no usar adjusted total-return;
- mantener cierre nominal;
- mostrar advertencia.

Para splits futuros:

- ajustar quantity;
- preservar valor económico;
- crear SPLIT_ADJUSTMENT;
- no generar retorno artificial.

Si aparece una acción corporativa no clasificada:

- no certificar el snapshot afectado;
- marcar CORPORATE_ACTION_REVIEW_REQUIRED;
- no inventar continuidad.

==================================================
12. CONFIANZA HISTÓRICA
==================================================

Distinguir dos dimensiones:

A. PRICE_COVERAGE

- COMPLETE;
- PARTIAL;
- BLOCKED.

B. HISTORICAL_CONFIDENCE

- CERTIFIED;
- PARTIAL_HISTORICAL_CONFIDENCE;
- BLOCKED.

Las 46 posiciones conocidas pueden contribuir a:

CERTIFIED

Las 36 posiciones con requested_notional desconocido contribuyen a:

PARTIAL_HISTORICAL_CONFIDENCE

La cartera/sesión debe recibir:

PARTIAL_HISTORICAL_CONFIDENCE

si existe al menos una posición activa relevante con esa confianza.

No excluir automáticamente esas posiciones del cálculo porque:

- formaron parte de la composición operada;
- excluirlas distorsionaría el Track.

Pero:

- no presentarlas como historia totalmente certificada;
- mostrar advertencia;
- conservar conteo y motivo.

Campos de snapshot requeridos o equivalentes:

- historical_confidence;
- partial_confidence_position_count;
- manual_review_required_count;
- confidence_reasons.

Si el schema actual no contiene estos campos:

- crear migración v28 únicamente si es estrictamente necesaria;
- debe ser aditiva;
- limitada al Track Papel;
- no incluir catalizadores;
- no reutilizar v28 de catalyst lifecycle;
- si existe reserva previa de versión, usar la siguiente versión libre;
- documentar la decisión.

Preferir un payload/read model derivado si puede representarse sin perder
persistencia crítica.

==================================================
13. SNAPSHOT DIARIO
==================================================

Crear exactamente un paper_track_snapshot por:

paper_portfolio_id
+ methodology_id
+ valuation_session

Campos mínimos:

- portfolio;
- methodology;
- valuation_session;
- paper_nav;
- paper_market_value;
- paper_cash;
- paper_units;
- paper_unit_value;
- daily_return;
- cumulative_return;
- external_flow;
- active_economic_positions;
- benchmark_nav;
- benchmark_units;
- benchmark_unit_value;
- benchmark_daily_return;
- benchmark_cumulative_return;
- drawdown;
- max_drawdown;
- price_coverage_status;
- historical_confidence;
- partial_confidence_position_count;
- certification_state;
- price_manifest_reference;
- calculated_at;
- content_hash.

No almacenar `0` para valores desconocidos.

Utilizar:

- NULL;
- estado explícito;
- warning.

==================================================
14. DRAWDOWN
==================================================

Por sesión:

running_peak_unit_value =
máximo paper_unit_value observado hasta la sesión.

drawdown_t =
paper_unit_value_t / running_peak_unit_value - 1

max_drawdown_t =
mínimo drawdown observado hasta la sesión.

Convención:

- drawdown actual <= 0;
- máximo drawdown <= 0;
- primera sesión = 0.

No utilizar mdd = 0 como fallback cuando falten datos.

Si la serie está incompleta:

- null;
- cobertura explícita.

==================================================
15. BENCHMARK SPY
==================================================

Política:

EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

Por cartera:

- SPY recibe el mismo flujo externo inicial;
- sesión igual al flujo de la cartera;
- cierre oficial SPY;
- benchmark unit value inicial = 100;
- benchmark units coherentes con el flujo;
- benchmark permanece invertido.

No replicar:

- compras internas;
- ventas internas;
- rebalanceos;
- reasignaciones;
- paper cash;
- cambios de membership.

Para futuras aportaciones o retiradas externas:

- replicar mismo importe;
- misma sesión;
- emisión o retirada neutral de unidades benchmark.

No usar puntos SPY anteriores a la inception de la cartera.

No usar los puntos extra devueltos por el proveedor si no son requeridos.

Calcular:

- benchmark NAV;
- benchmark unit value;
- daily return;
- cumulative return.

El diferencial puede exponerse:

paper cumulative return
- benchmark cumulative return

No denominarlo alfa estadística salvo metodología adicional.

==================================================
16. BACKFILL DE SNAPSHOTS
==================================================

Antes de escribir:

- reconstruir toda la serie en LAB;
- comparar identidades;
- generar preview;
- verificar cobertura;
- verificar confianza;
- verificar cash;
- verificar benchmark;
- verificar drawdown;
- verificar primera y última sesión.

Por cartera entregar preview:

- first session;
- last session;
- snapshot count;
- initial NAV;
- final NAV;
- initial unit value;
- final unit value;
- cumulative return;
- max drawdown;
- final cash;
- active positions;
- benchmark final value;
- benchmark return;
- confidence;
- warnings.

No escribir si:

- existe cash negativo fuera de tolerancia;
- falta un cierre;
- existe acción corporativa bloqueada;
- existe doble snapshot;
- NAV no reconcilia;
- la primera sesión no empieza en unit value 100;
- la aportación inicial genera retorno;
- una transacción al cierre genera retorno artificial.

==================================================
17. PERSISTENCIA TRANSACCIONAL
==================================================

Antes de Producción:

- backup;
- SHA-256;
- abrir backup;
- integrity_check;
- foreign_key_check;
- baseline completo;
- preview LAB.

Persistir el backfill de snapshots dentro de una transacción.

Si falla:

- rollback completo;
- paper_track_snapshot vuelve a cero;
- datos estructurales de Gate 2C-2 permanecen intactos;
- schema permanece v27 o en la versión aditiva necesaria.

La segunda ejecución debe:

- ser idempotente;
- no duplicar;
- no alterar hashes;
- devolver ALREADY_CURRENT o equivalente.

==================================================
18. DAILY-CLOSE
==================================================

No añadir carteras cat:* al camino del Book real.

Crear un adaptador explícito:

persistPaperTrackPortfolio

Responsabilidades:

1. resolver última sesión cerrada;
2. comprobar si existe snapshot;
3. cargar composición EOD efectiva;
4. obtener cierres oficiales;
5. validar cobertura;
6. calcular snapshot;
7. calcular benchmark;
8. persistir idempotentemente;
9. registrar warnings;
10. no tocar Book/cash/NAV reales.

Extender el orquestador diario existente para invocar por separado:

- persistencia real;
- persistencia papel.

No crear un segundo scheduler.

No llamar ciegamente a:

persistirTrackCartera

si presupone Book real o metodología de coste.

El fallo de una cartera papel:

- no debe impedir el cierre de las demás;
- no debe impedir el Track real;
- debe quedar registrado;
- debe reintentarse mediante catch-up.

==================================================
19. STARTUP CATCH-UP
==================================================

Al arrancar:

- resolver última sesión cerrada;
- detectar sesiones papel faltantes;
- reconstruirlas en orden;
- no duplicar;
- no depender del navegador;
- no crear posiciones;
- no activar hard cap;
- no formular tesis.

Si existe cobertura insuficiente:

- no usar precio live;
- no saltar silenciosamente;
- registrar PRICE_COVERAGE_PARTIAL;
- dejar la sesión pendiente;
- reintentar posteriormente.

==================================================
20. ENDPOINTS
==================================================

Crear un contrato normalizado para Papel.

Endpoint recomendado:

GET /paper-track/:portfolioId

o adaptar el endpoint existente sin romper consumidores.

Payload mínimo:

- portfolio_id;
- portfolio_kind = PAPER;
- methodology = PRICE_RETURN_TWR;
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
- membership_count;
- warnings;
- certification_state.

Cada punto de series:

- session;
- nav;
- unit_value;
- daily_return;
- cumulative_return;
- cash;
- market_value;
- drawdown;
- benchmark_unit_value;
- benchmark_return;
- coverage;
- confidence.

No mezclar el payload papel con:

- cash real;
- NAV real;
- market value real;
- Track de Crecimiento.

Mantener compatibilidad temporal con el endpoint anterior si la UI todavía lo
consume.

No modificar aún la presentación visual.

==================================================
21. MÉTRICAS
==================================================

Calcular:

- Paper NAV;
- Paper Index / unit value;
- retorno diario;
- retorno acumulado;
- retorno desde inicio;
- drawdown actual;
- máximo drawdown;
- volatilidad, si puede reutilizarse sin alterar metodología;
- benchmark return;
- diferencial frente a SPY;
- primera sesión;
- última sesión;
- sesiones valoradas;
- posiciones económicas activas;
- cobertura;
- confianza histórica.

No calcular o denominar:

- total return;
- alpha estadística;
- Sharpe;
- métricas no soportadas;

sin una metodología explícita.

==================================================
22. AISLAMIENTO DEL TRACK REAL
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
- hashes.

Resultado obligatorio:

GROWTH_TRACK_BYTE_IDENTICAL = PASS

Si el timestamp de respuesta impide igualdad byte a byte:

- comparar payload económico normalizado;
- explicar campos no deterministas;
- exigir igualdad económica exacta.

También:

REAL_BOOK_UNCHANGED = PASS
REAL_CASH_UNCHANGED = PASS
REAL_NAV_UNCHANGED = PASS
REAL_VALUATIONS_UNCHANGED = PASS

==================================================
23. PRUEBAS
==================================================

Crear:

verify-paper-track-engine.mjs
verify-paper-track-unitization.mjs
verify-paper-track-daily-series.mjs
verify-paper-track-benchmark.mjs
verify-paper-track-confidence.mjs
verify-paper-track-drawdown.mjs
verify-paper-track-daily-close.mjs
verify-paper-track-startup-catchup.mjs
verify-paper-track-endpoint.mjs
verify-paper-track-isolation.mjs

Casos obligatorios:

A. Primera sesión = unit value 100.
B. Aportación inicial neutral.
C. Compra interna neutral.
D. Rebalanceo neutral al cierre.
E. Reasignación pre-fill sin exposición en origen.
F. Movimiento +5 % produce +5 % sobre capital expuesto según NAV.
G. Paper cash forma parte del NAV.
H. Cash negativo bloqueado.
I. Retorno diario correcto.
J. Retorno acumulado correcto.
K. Drawdown correcto.
L. Benchmark desde inception.
M. Benchmark ignora operaciones internas.
N. Dividend warning.
O. Split neutral.
P. 36 posiciones generan partial historical confidence.
Q. Precio ausente no usa fallback.
R. Snapshot idempotente.
S. Daily-close no toca Track real.
T. Startup catch-up no duplica.
U. Endpoint no mezcla patrimonio real.
V. Crecimiento sin regresión.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- suites v27;
- suites Gate 2C-2;
- financial core;
- Growth Track;
- daily-close;
- repository integrity.

==================================================
24. DESPLIEGUE
==================================================

Antes:

- backup;
- integrity_check;
- foreign_key_check;
- baseline;
- preview LAB;
- tests verdes.

Después:

- una sola instancia;
- health;
- env-info;
- integrity_check;
- foreign_key_check;
- snapshots completos;
- endpoint operativo;
- daily-close integrado;
- catch-up idempotente;
- hard cap inactivo;
- legacy intacto;
- Crecimiento intacto.

No realizar todavía cambios visuales.

==================================================
25. CÓDIGO Y COMMIT
==================================================

Añadir:

- motor papel;
- backfill de snapshots;
- adaptador daily-close;
- startup catch-up;
- endpoints;
- read model;
- pruebas;
- documentación.

No añadir:

- scratchpad;
- staging;
- backups;
- bases;
- secretos;
- manifiestos con datos productivos innecesarios.

Ejecutar secret scan.

Crear un commit local:

feat(lens): add unitized daily paper track engine

No crear tag.
No hacer push.

==================================================
26. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = 66059a9
CANONICAL_HEAD_AFTER = <hash>

SCHEMA_BEFORE = <versión>
SCHEMA_AFTER = <versión>

PAPER_TRACK_METHOD =
PRICE_RETURN_TWR

PAPER_PORTFOLIOS_PROCESSED = <número>/5
PAPER_TRACK_SNAPSHOTS_CREATED = <número>
PAPER_FIRST_SESSION_BY_PORTFOLIO = <tabla>
PAPER_LAST_SESSION_BY_PORTFOLIO = <tabla>

PAPER_INITIAL_UNIT_VALUE = 100
PAPER_FINAL_UNIT_VALUE_BY_PORTFOLIO = <tabla>
PAPER_CUMULATIVE_RETURN_BY_PORTFOLIO = <tabla>
PAPER_MAX_DRAWDOWN_BY_PORTFOLIO = <tabla>
PAPER_FINAL_NAV_BY_PORTFOLIO = <tabla>
PAPER_FINAL_CASH_BY_PORTFOLIO = <tabla>

SPY_SERIES_CREATED = PASS/FAIL
SPY_FIRST_SESSION_BY_PORTFOLIO = <tabla>
SPY_FINAL_RETURN_BY_PORTFOLIO = <tabla>

PRICE_COVERAGE_COMPLETE = PASS/FAIL
HISTORICAL_CONFIDENCE_BY_PORTFOLIO = <tabla>
PARTIAL_CONFIDENCE_POSITION_COUNT = <número>
MANUAL_REVIEW_REQUIRED_COUNT = 36

PRE_FILL_REASSIGNMENT_ZERO_ORIGIN_EXPOSURE = PASS/FAIL
SAME_SESSION_NETTING_PRESERVED = PASS/FAIL
NO_ARTIFICIAL_TURNOVER = PASS/FAIL

DAILY_CLOSE_PAPER_ADAPTER = PASS/FAIL
STARTUP_CATCHUP = PASS/FAIL
PAPER_TRACK_ENDPOINT = <ruta>
PAPER_TRACK_ENDPOINT_GREEN = PASS/FAIL

SNAPSHOT_BACKFILL_TRANSACTIONAL = PASS/FAIL
SNAPSHOT_BACKFILL_IDEMPOTENT = PASS/FAIL

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

NO_POLICY_ACTIVATED = PASS/FAIL
GROWTH_TRACK_UNCHANGED = PASS/FAIL
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

Detenerse después del Gate 2D.