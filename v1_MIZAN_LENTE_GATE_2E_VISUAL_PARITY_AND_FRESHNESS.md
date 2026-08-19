MIZAN · LA LENTE
GATE 2E · FRESCURA OPERATIVA Y PARIDAD VISUAL
MANIFIESTO VERSIONADO · TRACKRECORDVIEW COMPARTIDO · CHROME

NATURALEZA

Esta ejecución autoriza:

SUBGATE 2E-A

1. completar la ingesta diaria de cierres oficiales para Papel;
2. ampliar el manifiesto de precios de forma versionada;
3. permitir que daily-close y startup catch-up avancen después del
   24 de julio de 2026;
4. probar la actualización cronológica e idempotente de snapshots.

SUBGATE 2E-B

5. crear o consolidar TrackRecordView;
6. alimentar la vista papel desde los endpoints canónicos;
7. igualar visualmente Papel con el Track Record de Crecimiento;
8. mostrar metodología, confianza y cobertura;
9. retirar papelChartHTML únicamente después de certificar paridad;
10. desplegar y validar en Chrome;
11. crear un único commit local.

No autoriza:

- activar MANUAL_NOTIONAL_HARD_CAP;
- fijar effective_from de la política prospectiva;
- cambiar el comportamiento de incorporación de acciones;
- crear aportaciones o retiradas;
- modificar posiciones papel;
- modificar holdings legacy;
- modificar snapshots legacy;
- modificar cat_composicion_log;
- modificar tesis;
- modificar catalizadores;
- modificar Campo de Caza;
- cambiar datos o metodología de Crecimiento;
- cambiar Book, cash o NAV reales;
- conectar con Wio;
- crear tag;
- hacer push;
- realizar operaciones Git remotas.

Gate 2E debe detenerse si 2E-A no queda verde.

No desplegar la nueva vista sobre una serie incapaz de actualizarse
diariamente.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = a201203
schema esperado = v29
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Estado confirmado:

PAPER_PORTFOLIOS = 5
PAPER_TRACK_SNAPSHOTS = 54

CANONICAL_RECONSTRUCTION_VERSION = 1
SINGLE_CANONICAL_RECONSTRUCTION = true
LEGACY_TRANSACTIONS_EXCLUDED_FROM_TRACK = true

ORIGINAL_MANIFEST_END_SESSION =
2026-07-24

ORIGINAL_MANIFEST_VERSION =
1

ORIGINAL_MANIFEST_HASH =
3c5bf06f68db1021a491dc511470eb1a29b0c8be81bf53165ee3c1ab21cc943a

RETURN_METHOD =
PRICE_RETURN_TWR

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

BENCHMARK =
SPY

BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

Confianza:

FULL_CONFIDENCE_SESSIONS = 15
PARTIAL_CONFIDENCE_SESSIONS = 39
BLOCKED_SESSIONS = 0
MANUAL_REVIEW_REQUIRED_OBJECTS = 36

Endpoint canónico:

GET /lens/paper-track/:portfolioId

Endpoints auxiliares:

GET /lens/paper-track/:portfolioId/summary
GET /lens/paper-track/:portfolioId/episodes
GET /lens/paper-track/:portfolioId/confidence

Política prospectiva:

MANUAL_NOTIONAL_HARD_CAP.status =
APPROVED_PENDING_ACTIVATION

MANUAL_NOTIONAL_HARD_CAP.effective_from =
NULL

Debe permanecer así.

==================================================
1. PRECONDICIONES
==================================================

Antes de modificar:

- confirmar HEAD a201203;
- confirmar schema v29;
- confirmar env-info;
- confirmar una sola raíz Git;
- inventariar untracked;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- confirmar una sola instancia;
- confirmar health;
- validar reconstruction version 1;
- validar las 54 filas de paper_track_snapshot;
- validar los endpoints read-only;
- validar hard cap sin effective_from.

Capturar baseline completo de Crecimiento:

- HTML/DOM del Track;
- captura de escritorio;
- captura móvil;
- payload;
- serie;
- métricas;
- ventanas;
- benchmark;
- drawdown;
- estados de cobertura;
- estados de carga/error;
- hashes económicos.

Capturar baseline de Papel:

- papelChartHTML;
- captura de las cinco carteras;
- payloads de endpoints;
- KPIs;
- warnings;
- primera y última sesión;
- series y benchmark.

No continuar si:

- existe una reconstrucción canónica distinta de 1;
- falta algún snapshot actual;
- existe sesión bloqueada;
- hard cap está activo;
- integrity_check o foreign_key_check falla.

==================================================
2. RESOLUCIÓN DINÁMICA DE LA ÚLTIMA SESIÓN
==================================================

En cada daily-close y startup catch-up resolver:

candidate_end_session =
última sesión NYSE completamente cerrada según America/New_York.

Después comprobar:

- que el proveedor canónico ofrece cierre EOD;
- que el precio corresponde exactamente a la sesión;
- que todos los tickers requeridos tienen cobertura;
- que SPY tiene cobertura.

La sesión final certificable será:

coverage_end_session =
última sesión consecutiva completamente cubierta.

No codificar:

2026-07-24

No utilizar:

- fecha UTC directamente;
- fecha de Dubái como sesión;
- precio live;
- apertura;
- previous close como sustituto;
- adjusted total-return;
- interpolación.

Regla temporal para el 27 de julio de 2026:

- si la ejecución ocurre antes del cierre de Nueva York, la última sesión
  cerrada sigue siendo el 24 de julio de 2026;
- si ocurre después del cierre y los cierres EOD del 27 de julio están
  disponibles y validados, la serie debe poder extenderse al 27 de julio;
- si el mercado cerró pero el proveedor aún no publicó todos los cierres,
  mantener el 24 de julio y reintentar posteriormente.

==================================================
3. MANIFIESTO DE PRECIOS VERSIONADO
==================================================

No modificar el manifiesto v1 in-place.

Cuando existan sesiones nuevas:

1. identificar sesiones faltantes;
2. determinar tickers económicamente activos por cartera y sesión;
3. añadir SPY;
4. deduplicar por ticker + sesión + price basis + provider;
5. obtener cierres mediante el proveedor canónico;
6. validar sesiones y símbolos;
7. clasificar acciones corporativas;
8. crear una nueva versión del manifiesto;
9. calcular un nuevo hash;
10. persistir evidencia durable;
11. generar snapshots únicamente tras validar el manifiesto nuevo.

El manifiesto nuevo debe referenciar:

- previous_manifest_hash;
- manifest_version;
- created_at;
- first_added_session;
- last_added_session;
- provider;
- price_basis;
- added_price_points;
- validation status;
- content hash.

No volver a descargar ni cambiar silenciosamente los puntos históricos ya
sellados.

Si el proveedor devuelve un valor diferente para un punto ya sellado:

- no sobrescribirlo;
- registrar PRICE_REVISION_DETECTED;
- mantener el snapshot canónico;
- exigir una reconstrucción versionada separada si la revisión es material.

==================================================
4. INGESTA FAIL-CLOSED
==================================================

La ampliación del manifiesto debe ocurrir únicamente en:

- daily-close;
- startup catch-up;
- una herramienta interna explícita de reconstrucción.

Nunca en:

- GET /lens/paper-track/*;
- render de UI;
- apertura de una pestaña;
- interacción del navegador.

Si falta un precio obligatorio:

- no generar snapshot parcial de la sesión;
- no saltar a una sesión posterior;
- no usar fallback;
- conservar el último snapshot válido;
- registrar PRICE_COVERAGE_INCOMPLETE;
- reintentar posteriormente.

El fallo de una cartera papel:

- no debe impedir el Track real;
- no debe impedir el procesamiento de otras carteras papel;
- debe quedar registrado por cartera y sesión.

==================================================
5. PRUEBA DE ACTUALIZACIÓN FUTURA
==================================================

Probar en LAB, sin depender de que el mercado haya cerrado hoy:

A. UNA SESIÓN NUEVA COMPLETA

- extender manifiesto;
- crear snapshot;
- actualizar endpoint;
- mantener idempotencia.

B. SESIÓN CERRADA SIN PRECIO

- no extender;
- no crear snapshot;
- reintento pendiente.

C. PRECIO FALTANTE EN UN TICKER

- bloquear esa cartera;
- no usar live;
- otras carteras pueden continuar si su cobertura es completa.

D. PRECIO FALTANTE EN SPY

- no certificar el snapshot de esa cartera;
- no inventar benchmark.

E. SEGUNDA EJECUCIÓN

- no duplicar evidencia;
- no duplicar snapshots;
- no cambiar hashes.

F. REVISIÓN DEL PROVEEDOR

- no sobrescribir evidencia histórica;
- alertar PRICE_REVISION_DETECTED.

G. EARLY CLOSE

- utilizar el cierre específico de la sesión.

Declarar:

VERSIONED_MANIFEST_EXTENSION = PASS/FAIL
FUTURE_SESSION_CATCHUP = PASS/FAIL
FAIL_CLOSED_PRICE_INGESTION = PASS/FAIL
HISTORICAL_PRICE_IMMUTABILITY = PASS/FAIL

Si cualquiera falla:

- detener Gate 2E;
- no cambiar la UI.

==================================================
6. CONTRATO NORMALIZADO DE TRACKRECORDVIEW
==================================================

Crear o consolidar un componente compartido:

TrackRecordView

Debe recibir un read model normalizado, no acceder directamente a tablas ni
calcular metodología económica en la UI.

Contrato mínimo:

- portfolio_id;
- portfolio_kind;
- title;
- methodology;
- income_policy;
- benchmark;
- first_session;
- last_session;
- as_of;
- series;
- benchmark_series;
- metrics;
- windows;
- risk;
- drawdown;
- coverage;
- confidence;
- warnings;
- loading_state;
- error_state;
- empty_state;
- disclaimer.

La UI no debe recalcular:

- NAV;
- TWR;
- benchmark;
- drawdown;
- confianza;
- cobertura.

Debe mostrar los valores entregados por el endpoint.

==================================================
7. ADAPTADOR DE CRECIMIENTO
==================================================

Crecimiento debe utilizar TrackRecordView mediante un adaptador que preserve
su payload y metodología actual.

No modificar:

- posicionPnL;
- valorEnFecha;
- valuations;
- retorno sobre coste;
- benchmark;
- ventanas;
- métricas;
- drawdown;
- datos históricos;
- endpoints;
- resultados.

Requisito:

GROWTH_TRACK_DATA_UNCHANGED = PASS

La extracción del componente no puede cambiar:

- orden visual;
- textos existentes;
- escalas;
- tooltips;
- selectores;
- valores;
- responsive.

Si hacer compartido el componente altera Crecimiento:

- corregir el adaptador;
- no aceptar una regresión bajo el argumento de refactor.

==================================================
8. ADAPTADOR PAPEL
==================================================

Papel debe consumir exclusivamente:

GET /lens/paper-track/:portfolioId

Los endpoints auxiliares pueden usarse para detalles, pero no deben ser
necesarios para dibujar la serie principal si el payload canónico ya incluye
la información.

Mapear:

- unit_value → serie principal;
- cumulative_return → retorno;
- benchmark_unit_value → serie SPY;
- benchmark_cumulative_return → benchmark;
- paper_nav → Paper NAV;
- paper_cash → cash papel;
- paper_market_value → exposición;
- drawdown → drawdown;
- max_drawdown → máximo drawdown;
- historical_confidence → confianza;
- price coverage → cobertura.

No usar:

- endpoint legacy;
- holdings actuales;
- snapshots legacy;
- cálculo de coste medio;
- papelChartHTML como fuente económica.

==================================================
9. PARIDAD VISUAL
==================================================

Crecimiento y Papel deben compartir:

- contenedor;
- cabecera;
- estructura del gráfico;
- SVG o renderer;
- selector temporal;
- ejes;
- escalas;
- colores estructurales;
- grid;
- leyenda;
- tooltips;
- indicadores de benchmark;
- sección de drawdown;
- tarjetas KPI;
- estados de carga;
- estados de error;
- estado vacío;
- cobertura;
- responsive;
- tipografía;
- formato de fechas;
- formato de porcentajes;
- formato monetario;
- accesibilidad básica.

Selectores temporales:

- 1D;
- MTD;
- YTD;
- 1Y;

o exactamente los existentes en Crecimiento.

Si una ventana no tiene historia suficiente:

- no fabricar puntos;
- mostrar la parte disponible;
- indicar fecha de inception cuando proceda.

==================================================
10. DIFERENCIAS VISIBLES DE PAPEL
==================================================

Papel debe mostrar claramente:

- etiqueta PAPEL;
- nombre de cartera;
- PRICE_RETURN_TWR;
- PAPER NAV o PAPER INDEX;
- benchmark SPY;
- INCOME_NOT_INCLUDED;
- disclaimer de simulación;
- primera sesión;
- última sesión;
- número de sesiones;
- posiciones activas;
- paper cash;
- cobertura;
- confianza histórica.

No denominarlo:

- rentabilidad real;
- ejecución real;
- total return;
- alpha estadística;
- Track de Crecimiento.

No mezclar cifras papel con:

- patrimonio real;
- cash real;
- NAV real;
- Book real;
- sizing real.

==================================================
11. CONFIANZA HISTÓRICA VISIBLE
==================================================

Cuando una ventana contenga sesiones parciales, mostrar:

PARTIAL_HISTORICAL_CONFIDENCE

Texto conceptual:

“Parte del historial fue reconstruida con composición y precios
point-in-time, pero el importe solicitado original de algunas posiciones no
está disponible.”

Mostrar:

- número de objetos manual-review;
- número de sesiones parciales;
- motivo;
- enlace o expansión a detalles si existe.

No afirmar:

- FULLY_CERTIFIED;
- historial completo exacto;
- requested_notional conocido.

Para sesiones con confianza completa:

FULL_HISTORICAL_CONFIDENCE

La advertencia debe corresponder a la ventana seleccionada, no solo al estado
global de la cartera.

==================================================
12. COBERTURA Y FRESCURA
==================================================

Mostrar:

- última sesión certificada;
- estado de cobertura;
- precio basis;
- proveedor, cuando corresponda;
- as_of.

Estados:

- CURRENT;
- PRICE_COVERAGE_INCOMPLETE;
- CATCHUP_PENDING;
- BLOCKED;
- PARTIAL_HISTORICAL_CONFIDENCE.

Si la última sesión cerrada es posterior a last_session:

- no presentar la curva como actual;
- mostrar CATCHUP_PENDING o PRICE_COVERAGE_INCOMPLETE;
- conservar el último punto válido.

No usar datos live dentro de la curva certificada.

==================================================
13. TOOLTIP
==================================================

El tooltip de Papel debe mostrar como mínimo:

- sesión;
- Paper Unit Value;
- retorno acumulado;
- retorno diario;
- Paper NAV;
- SPY;
- diferencial simple;
- drawdown;
- confianza;
- cobertura.

No mostrar valores desconocidos como cero.

Usar:

- “No disponible”;
- warning;
- estado explícito.

==================================================
14. KPIS
==================================================

Mostrar para Papel:

- Paper NAV;
- Paper Unit Value;
- retorno acumulado;
- retorno diario de la última sesión;
- SPY acumulado;
- diferencial frente a SPY;
- drawdown actual;
- máximo drawdown;
- paper cash;
- exposición;
- posiciones activas;
- primera sesión;
- última sesión;
- sesiones valoradas;
- cobertura;
- confianza.

No denominar el diferencial “alpha”.

No mostrar Sharpe, volatilidad u otras métricas si no se encuentran
formalmente soportadas por el endpoint y la metodología.

==================================================
15. RETIRADA DE PAPELCHARTHTML
==================================================

No borrar papelChartHTML al inicio.

Proceso:

1. implementar TrackRecordView;
2. adaptar Crecimiento;
3. adaptar Papel;
4. validar datos;
5. validar screenshots;
6. validar Chrome;
7. validar responsive;
8. validar estados;
9. comprobar cero regresiones;
10. dejar papelChartHTML fuera de uso;
11. eliminarlo únicamente si no tiene otros consumidores.

Si aún existe algún consumidor:

- marcarlo deprecated;
- no mantener dos caminos activos para la misma vista.

Resultado final:

PAPER_TRACK_RENDERER =
TrackRecordView

==================================================
16. ESTADOS DE CARGA Y ERROR
==================================================

Validar:

- loading;
- endpoint timeout;
- cartera inexistente;
- serie vacía;
- cobertura incompleta;
- confianza parcial;
- catch-up pendiente;
- proveedor no disponible;
- error interno;
- última sesión válida disponible.

La UI no debe:

- quedarse en blanco;
- mostrar cero como fallback;
- ejecutar fetch de precios;
- generar snapshots;
- modificar estado.

==================================================
17. CHROME Y CAPTURAS
==================================================

Validar en Chrome real:

A. CRECIMIENTO

- escritorio;
- móvil;
- cada selector temporal;
- tooltip;
- drawdown;
- benchmark;
- estados de carga/error.

B. PAPEL

- las cinco carteras;
- escritorio;
- móvil;
- cada selector;
- tooltip;
- drawdown;
- SPY;
- confianza;
- cobertura;
- etiqueta y disclaimer.

Comparar capturas:

- Crecimiento antes/después;
- Papel antes/después;
- desktop;
- mobile.

Crecimiento debe permanecer visualmente idéntico salvo diferencias
estrictamente necesarias y aprobadas para compartir el componente.

No aceptar:

- desplazamientos de layout;
- escalas distintas;
- tooltips incompletos;
- drawdown omitido;
- mdd=0 falso;
- overflow móvil;
- errores JavaScript.

==================================================
18. ENDPOINTS READ-ONLY
==================================================

Confirmar que los GET continúan siendo read-only.

Las acciones de UI:

- cambio de ventana;
- hover;
- cambio de cartera;
- recarga;

no pueden producir:

- INSERT;
- UPDATE;
- DELETE;
- fetch de precios;
- creación de snapshots;
- activación de políticas.

Medir contadores antes/después de navegar por la UI.

Resultado:

UI_NAVIGATION_DATABASE_WRITES = 0

==================================================
19. AISLAMIENTO DE CRECIMIENTO Y DATOS REALES
==================================================

Antes y después comparar:

- payload económico de Crecimiento;
- serie;
- métricas;
- ventanas;
- benchmark;
- drawdown;
- valuations;
- holdings;
- movimientos;
- Book;
- cash;
- NAV.

Resultados obligatorios:

GROWTH_TRACK_DATA_UNCHANGED = PASS
GROWTH_TRACK_VISUAL_PARITY_PRESERVED = PASS
REAL_BOOK_UNCHANGED = PASS
REAL_CASH_UNCHANGED = PASS
REAL_NAV_UNCHANGED = PASS
REAL_VALUATIONS_UNCHANGED = PASS

También:

HOLDINGS_LEGACY_UNCHANGED = PASS
SNAPSHOTS_LEGACY_UNCHANGED = PASS
THESES_UNCHANGED = PASS
CATALYSTS_UNCHANGED = PASS
HUNTING_FIELD_UNCHANGED = PASS

==================================================
20. PRUEBAS
==================================================

Crear o consolidar:

verify-paper-price-manifest-extension.mjs
verify-paper-future-session-catchup.mjs
verify-paper-price-ingestion-fail-closed.mjs
verify-track-record-view-contract.mjs
verify-growth-track-view-adapter.mjs
verify-paper-track-view-adapter.mjs
verify-paper-track-confidence-ui.mjs
verify-paper-track-coverage-ui.mjs
verify-paper-track-ui-parity.mjs
verify-paper-track-ui-readonly.mjs
verify-paper-track-responsive.mjs
verify-paper-track-chrome.mjs
verify-paper-track-isolation-ui.mjs

Casos obligatorios:

A. Manifiesto v1 permanece inmutable.
B. Nueva sesión crea nueva versión.
C. Sesión sin cobertura no crea snapshot.
D. GET no obtiene precios.
E. GET no crea snapshot.
F. Crecimiento usa mismo componente sin cambiar datos.
G. Papel usa endpoint canónico.
H. Papel muestra PRICE_RETURN_TWR.
I. Papel muestra INCOME_NOT_INCLUDED.
J. Confianza parcial visible.
K. Confianza por ventana.
L. Cobertura y frescura visibles.
M. SPY visible.
N. Drawdown visible.
O. Selectores funcionan.
P. Tooltip completo.
Q. Móvil sin overflow.
R. Estados de error.
S. Navegación UI = cero escrituras.
T. Hard cap sigue inactivo.
U. Crecimiento sin regresión.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- suites v27/v28/v29;
- suites Gate 2C-2;
- suites Gate 2D-1;
- suites Gate 2D-2;
- financial core;
- Growth Track;
- daily-close;
- startup catch-up;
- repository integrity.

==================================================
21. DESPLIEGUE
==================================================

SUBGATE 2E-A debe quedar verde antes de desplegar 2E-B.

Antes:

- backup;
- SHA-256;
- integrity_check;
- foreign_key_check;
- baseline;
- capturas;
- tests;
- git limpio.

Después:

- una sola instancia;
- health;
- env-info;
- manifest extension operativo;
- daily-close capaz de avanzar;
- endpoint actualizado;
- TrackRecordView activo;
- Papel visible;
- Crecimiento intacto;
- Chrome verde;
- hard cap inactivo.

No activar MANUAL_NOTIONAL_HARD_CAP.

==================================================
22. CÓDIGO Y COMMIT
==================================================

Añadir:

- extensión versionada del manifiesto;
- pipeline de ingesta diaria;
- TrackRecordView;
- adaptador de Crecimiento;
- adaptador de Papel;
- estados de confianza/cobertura;
- pruebas;
- documentación.

No añadir:

- hard-cap funcional;
- UI de nuevas asignaciones;
- scratchpad;
- staging;
- backups;
- bases;
- secretos;
- datos productivos innecesarios.

Ejecutar secret scan.

Crear un único commit local:

feat(lens): add shared paper track view and daily price freshness

No crear tag.
No hacer push.

==================================================
23. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = a201203
CANONICAL_HEAD_AFTER = <hash>

SCHEMA_BEFORE = v29
SCHEMA_AFTER = <v29/NEXT_AVAILABLE_SCHEMA_VERSION>

ORIGINAL_MANIFEST_VERSION = 1
CURRENT_MANIFEST_VERSION = <número>
ORIGINAL_MANIFEST_HASH =
3c5bf06f68db1021a491dc511470eb1a29b0c8be81bf53165ee3c1ab21cc943a

CURRENT_MANIFEST_HASH = <hash>
CURRENT_COVERAGE_END_SESSION = <fecha>

VERSIONED_MANIFEST_EXTENSION = PASS/FAIL
FUTURE_SESSION_CATCHUP = PASS/FAIL
FAIL_CLOSED_PRICE_INGESTION = PASS/FAIL
HISTORICAL_PRICE_IMMUTABILITY = PASS/FAIL

PAPER_TRACK_RENDERER =
TrackRecordView

TRACK_RECORD_VIEW_SHARED = PASS/FAIL
PAPER_VISUAL_PARITY_WITH_GROWTH = PASS/FAIL
GROWTH_VISUAL_UNCHANGED = PASS/FAIL
GROWTH_TRACK_DATA_UNCHANGED = PASS/FAIL

PAPER_PORTFOLIOS_VISIBLE = <número>/5
PAPER_CURVES_VISIBLE = <número>/5
PAPER_SPY_VISIBLE = <número>/5
PAPER_DRAWDOWN_VISIBLE = <número>/5

PARTIAL_HISTORICAL_CONFIDENCE_VISIBLE = PASS/FAIL
INCOME_NOT_INCLUDED_VISIBLE = PASS/FAIL
PRICE_COVERAGE_VISIBLE = PASS/FAIL
LAST_CERTIFIED_SESSION_VISIBLE = PASS/FAIL
PAPER_SIMULATION_DISCLAIMER_VISIBLE = PASS/FAIL

TEMPORAL_SELECTORS_GREEN = PASS/FAIL
TOOLTIPS_GREEN = PASS/FAIL
RESPONSIVE_GREEN = PASS/FAIL
CHROME_GREEN = PASS/FAIL
JAVASCRIPT_ERRORS = <número>

UI_NAVIGATION_DATABASE_WRITES = 0
GET_ENDPOINTS_READ_ONLY = PASS/FAIL

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

HARD_CAP_ACTIVE =
FALSE

NO_POLICY_ACTIVATED = PASS/FAIL
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

EXACT_BLOCKERS_BEFORE_GATE_2F:
1. <bloqueo>
2. <bloqueo>
3. <bloqueo>

Detenerse después del Gate 2E.