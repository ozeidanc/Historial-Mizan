MIZAN · LA LENTE
GATE 2E-B · GRÁFICA DEL TRACK RECORD DE CARTERAS CATALIZADAS

OBJETIVO ÚNICO

Crear en La Lente una gráfica autónoma que muestre el track record histórico
de los stocks que formaron parte de cada cartera catalizada.

La gráfica debe utilizar exclusivamente el Track Papel ya calculado:

- PRICE_RETURN_TWR;
- Paper Unit Value;
- Paper NAV;
- benchmark SPY;
- drawdown;
- composiciones point-in-time;
- confianza histórica;
- cobertura de precios.

La gráfica debe tener un formato visual profesional equivalente al formato
solicitado por Omar:

- línea principal de rentabilidad;
- comparación con SPY;
- selectores temporales;
- ejes claros;
- tooltip;
- KPIs;
- drawdown;
- responsive;
- escritorio y móvil.

No existe ninguna relación funcional con la cartera de Crecimiento.

No inspeccionar, modificar, adaptar, compartir ni reutilizar código específico
de Crecimiento.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = 4565e22
schema esperado = v29
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Estado disponible:

PAPER_PORTFOLIOS = 5
PAPER_TRACK_SNAPSHOTS = 54

PAPER_TRACK_METHOD =
PRICE_RETURN_TWR

BENCHMARK =
SPY

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

FULL_CONFIDENCE_SESSIONS =
15

PARTIAL_CONFIDENCE_SESSIONS =
39

BLOCKED_SESSIONS =
0

MANUAL_REVIEW_REQUIRED_OBJECTS =
36

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

Debe permanecer inactiva.

==================================================
1. ALCANCE
==================================================

Crear una vista específica:

LensPaperTrackView

La vista debe representar por separado las cinco carteras catalizadas:

- cat:catalizada;
- cat:catalizada-2;
- cat:catalizada-2b;
- cat:catalizada-3;
- cat:catalizada-3b.

Al seleccionar una cartera, la gráfica debe representar su evolución
histórica real point-in-time.

Debe utilizar:

- los stocks económicamente activos en cada sesión;
- las cantidades certificadas vigentes;
- las composiciones EOD reconstruidas;
- los rebalanceos históricos AS_OPERATED;
- los cierres oficiales;
- Paper Cash;
- Paper NAV;
- Paper Unit Value.

No aplicar la composición actual hacia atrás.

No incluir un stock antes de su entrada económica.

No mantener un stock después de su salida o cambio de composición.

No atribuir exposición a la cartera de origen en las 39 reasignaciones
pre-fill.

==================================================
2. RESTRICCIONES
==================================================

No modificar ni utilizar como dependencia:

- cartera de Crecimiento;
- gráfica de Crecimiento;
- renderer de Crecimiento;
- endpoints de Crecimiento;
- datos de Crecimiento;
- CSS específico de Crecimiento;
- DOM de Crecimiento;
- listeners de Crecimiento;
- metodología de Crecimiento.

No crear:

- componente compartido con Crecimiento;
- adaptador de Crecimiento;
- comparación técnica con Crecimiento;
- dependencia entre ambos tracks.

No modificar:

- motor PRICE_RETURN_TWR;
- snapshots Papel;
- manifiesto de precios;
- daily-close;
- startup catch-up;
- endpoints económicos;
- holdings legacy;
- snapshots legacy;
- cat_composicion_log;
- tesis;
- catalizadores;
- Campo de Caza;
- Book real;
- cash real;
- NAV real;
- movimientos;
- decisiones;
- resizing.

No activar MANUAL_NOTIONAL_HARD_CAP.

No hacer push.
No crear tag.
No realizar operaciones Git remotas.

==================================================
3. FUENTE ÚNICA DE DATOS
==================================================

LensPaperTrackView debe consumir exclusivamente:

GET /lens/paper-track/:portfolioId

No calcular datos económicos en el frontend.

Mapeo:

- paper_unit_value:
  línea principal del Track;

- cumulative_return:
  rentabilidad acumulada;

- daily_return:
  rentabilidad diaria;

- paper_nav:
  valor económico de la cartera papel;

- paper_cash:
  efectivo papel;

- paper_market_value:
  exposición invertida;

- benchmark_unit_value:
  línea comparativa de SPY;

- benchmark_cumulative_return:
  retorno acumulado de SPY;

- drawdown:
  drawdown diario;

- max_drawdown:
  máximo drawdown;

- active_positions:
  número de stocks económicamente activos;

- historical_confidence:
  confianza histórica;

- coverage:
  cobertura de precios;

- last_session:
  última sesión certificada.

No utilizar como fuente:

- holdings actuales;
- snapshots legacy;
- precio de entrada legacy;
- coste medio;
- composición actual aplicada hacia atrás;
- cálculos hechos en el navegador.

==================================================
4. QUÉ REPRESENTA LA LÍNEA PRINCIPAL
==================================================

La línea principal debe representar:

PAPER_UNIT_VALUE

Base inicial:

100

Por tanto:

- 100,00 = valor inicial;
- 103,00 = +3 % desde inception;
- 97,00 = −3 % desde inception.

La línea refleja conjuntamente el resultado de los stocks que componían la
cartera en cada sesión, ponderados según la composición histórica
point-in-time.

No dibujar como línea principal:

- suma de precios;
- promedio simple de retornos;
- cantidad de stocks;
- NAV real;
- coste original;
- retorno de la composición actual hacia atrás.

La serie debe proceder directamente de paper_track_snapshot.

==================================================
5. BENCHMARK
==================================================

Mostrar una segunda línea:

SPY

La comparación debe comenzar en la inception session de cada cartera.

SPY debe utilizar:

BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

No replicar:

- incorporaciones internas;
- rebalanceos;
- reasignaciones;
- cambios de composición;
- Paper Cash.

La leyenda debe distinguir claramente:

- nombre de la cartera catalizada;
- SPY.

==================================================
6. ESTRUCTURA VISUAL
==================================================

La vista debe incluir:

A. CABECERA

- nombre de la cartera;
- etiqueta PAPEL;
- metodología PRICE_RETURN_TWR;
- última sesión certificada;
- estado de cobertura.

B. KPIS

- Paper NAV;
- Paper Unit Value;
- retorno acumulado;
- último retorno diario;
- retorno SPY;
- diferencial frente a SPY;
- drawdown actual;
- máximo drawdown;
- Paper Cash;
- exposición;
- stocks activos.

C. GRÁFICA PRINCIPAL

- línea de Paper Unit Value;
- línea SPY;
- ejes;
- grid;
- tooltip;
- leyenda;
- selector temporal.

D. DRAWDOWN

- sección inferior;
- drawdown diario;
- máximo drawdown;
- escala negativa clara.

E. ESTADOS Y AVISOS

- confianza histórica;
- cobertura;
- frescura;
- dividendos no incluidos;
- simulación papel.

==================================================
7. SELECTOR DE CARTERA
==================================================

Permitir seleccionar cualquiera de las cinco carteras.

Al cambiar de cartera:

1. solicitar su endpoint;
2. mostrar loading;
3. renderizar su serie;
4. renderizar SPY;
5. actualizar KPIs;
6. actualizar drawdown;
7. actualizar confianza y cobertura;
8. conservar el selector temporal cuando sea posible.

No mezclar en una misma curva principal varias carteras.

Cada cartera tiene:

- inception propia;
- composición propia;
- NAV propio;
- unidades propias;
- SPY propio desde inception;
- confianza propia.

==================================================
8. SELECTORES TEMPORALES
==================================================

Incluir:

- 1D;
- MTD;
- YTD;
- 1Y;
- MAX.

Dado que el historial actual es corto:

- no inventar puntos anteriores a inception;
- recortar la ventana al historial disponible;
- mostrar la fecha de inception;
- mantener visible el selector elegido;
- indicar cuando una ventana solo contiene parte del periodo solicitado.

MAX debe mostrar toda la historia disponible de la cartera.

==================================================
9. TOOLTIP
==================================================

Al mover el cursor o tocar la gráfica, mostrar:

- sesión;
- Paper Unit Value;
- retorno diario;
- retorno acumulado;
- Paper NAV;
- Paper Cash;
- exposición;
- stocks activos;
- SPY;
- retorno acumulado SPY;
- diferencial frente a SPY;
- drawdown;
- confianza;
- cobertura.

No mostrar valores desconocidos como cero.

Mostrar:

- “No disponible”;
- warning;
- estado explícito.

No denominar el diferencial frente a SPY:

alpha.

==================================================
10. KPIS
==================================================

Mostrar:

- Paper NAV;
- Paper Unit Value;
- retorno acumulado;
- retorno diario más reciente;
- retorno acumulado SPY;
- diferencial simple frente a SPY;
- drawdown actual;
- máximo drawdown;
- Paper Cash;
- exposición invertida;
- número de stocks activos;
- primera sesión;
- última sesión;
- sesiones valoradas.

No mostrar:

- NAV real;
- cash real;
- patrimonio real;
- sizing real;
- rentabilidad total con dividendos;
- métricas no soportadas.

==================================================
11. DRAWDOWN
==================================================

Mostrar una sección específica de drawdown.

Convenciones:

drawdown <= 0
max_drawdown <= 0

No mostrar:

mdd = 0

como fallback.

Si falta el dato:

- NULL;
- “No disponible”;
- estado explícito.

La ventana del drawdown debe corresponder al periodo visualizado.

==================================================
12. CONFIANZA HISTÓRICA
==================================================

Mostrar:

PARTIAL_HISTORICAL_CONFIDENCE

cuando la ventana visualizada incluya sesiones parciales.

Mensaje:

“Parte del historial fue reconstruida con composición y precios point-in-time,
pero el importe solicitado original de algunas posiciones no está
disponible.”

Mostrar, cuando esté disponible:

- sesiones parciales;
- objetos sujetos a revisión;
- motivo;
- MANUAL_REVIEW_REQUIRED.

No presentar esos tramos como historia completamente certificada.

Si toda la ventana tiene confianza completa:

FULL_HISTORICAL_CONFIDENCE

==================================================
13. COBERTURA Y FRESCURA
==================================================

Mostrar:

- última sesión NYSE cerrada;
- última sesión certificada;
- as_of;
- price basis;
- estado de cobertura.

Estados:

- CURRENT;
- CATCHUP_PENDING;
- PRICE_COVERAGE_INCOMPLETE;
- PARTIAL_HISTORICAL_CONFIDENCE;
- BLOCKED.

Si existe una sesión cerrada posterior a la última certificada:

- no presentar la serie como actual;
- mostrar CATCHUP_PENDING;
- mantener el último punto válido.

No utilizar datos live.

==================================================
14. DISCLAIMER
==================================================

Mostrar claramente:

“Simulación papel point-in-time. Rentabilidad por precio; dividendos no
incluidos.”

También:

- no representa ejecución real;
- no forma parte del patrimonio real;
- no forma parte del Book real.

No saturar la gráfica con texto.

Puede presentarse mediante:

- etiqueta;
- icono informativo;
- tooltip;
- pie compacto.

==================================================
15. ESTILO VISUAL
==================================================

Crear una gráfica limpia, profesional y consistente con el dashboard.

Debe incluir:

- fondo y contenedor coherentes;
- línea principal claramente visible;
- línea SPY diferenciada;
- grid discreto;
- ejes legibles;
- colores coherentes;
- tooltip compacto;
- KPIs ordenados;
- drawdown alineado;
- responsive;
- estados de carga y error claros.

La petición “como la gráfica que Omar indicó” significa reproducir su estilo
visual general:

- distribución;
- claridad;
- jerarquía;
- interacción;
- legibilidad.

No significa compartir código ni establecer dependencias con otra cartera.

==================================================
16. AISLAMIENTO TÉCNICO
==================================================

Utilizar namespace propio:

.lens-paper-track
.lens-paper-track__header
.lens-paper-track__kpis
.lens-paper-track__chart
.lens-paper-track__tooltip
.lens-paper-track__drawdown
.lens-paper-track__warning

Evitar:

- selectores globales;
- IDs reutilizados;
- listeners globales;
- variables globales mutables;
- CSS que afecte otras vistas.

Resultados:

PAPER_CSS_NAMESPACED = PASS
PAPER_DOM_ISOLATED = PASS

==================================================
17. RENDERER ANTERIOR
==================================================

Inventariar:

papelChartHTML

y cualquier renderer Papel anterior.

Proceso:

1. localizar consumidores;
2. implementar LensPaperTrackView;
3. conectar el endpoint canónico;
4. validar las cinco carteras;
5. validar todos los estados;
6. activar LensPaperTrackView;
7. dejar el renderer anterior inactivo;
8. eliminarlo solo si tiene cero consumidores.

No mantener dos renderers activos simultáneamente.

Estado final permitido:

REMOVED

o:

DEPRECATED_INACTIVE

==================================================
18. ESTADOS DE UI
==================================================

Validar:

- loading;
- timeout;
- cartera inexistente;
- serie vacía;
- endpoint 4xx;
- endpoint 5xx;
- cobertura incompleta;
- catch-up pendiente;
- confianza parcial;
- blocked;
- última sesión válida.

La UI no debe:

- quedar en blanco;
- mostrar ceros falsos;
- obtener precios;
- crear snapshots;
- ejecutar catch-up;
- activar políticas;
- escribir en la base.

==================================================
19. RESPONSIVE Y CHROME
==================================================

Validar en Chrome real.

Viewports:

- 1440 × 900;
- 1280 × 720;
- 390 × 844;
- 360 × 800.

Validar las cinco carteras en:

- MAX;
- 1D;
- MTD;
- YTD;
- 1Y.

Comprobar:

- curva;
- SPY;
- KPIs;
- tooltip;
- drawdown;
- confianza;
- cobertura;
- disclaimer;
- loading;
- error;
- empty.

No aceptar:

- overflow horizontal;
- clipping;
- superposición;
- ejes ilegibles;
- tooltip fuera del viewport;
- benchmark ausente;
- drawdown omitido;
- errores JavaScript.

==================================================
20. UI READ-ONLY
==================================================

Capturar contadores antes y después de:

- abrir las cinco carteras;
- cambiar ventanas;
- utilizar tooltips;
- recargar;
- cambiar de cartera.

Resultado:

UI_NAVIGATION_DATABASE_WRITES = 0

La navegación no puede:

- obtener precios;
- crear snapshots;
- ejecutar catch-up;
- activar políticas;
- crear solicitudes;
- modificar datos.

==================================================
21. PRUEBAS
==================================================

Crear:

verify-lens-paper-track-view.mjs
verify-lens-paper-track-data-source.mjs
verify-lens-paper-track-composition-pit.mjs
verify-lens-paper-track-benchmark-ui.mjs
verify-lens-paper-track-drawdown-ui.mjs
verify-lens-paper-track-confidence-ui.mjs
verify-lens-paper-track-coverage-ui.mjs
verify-lens-paper-track-readonly-ui.mjs
verify-lens-paper-track-responsive.mjs
verify-lens-paper-track-chrome.mjs

Casos:

A. Las cinco carteras están disponibles.
B. Cada cartera tiene su propia curva.
C. La curva usa Paper Unit Value.
D. La composición es point-in-time.
E. Un stock no aparece antes de su entrada.
F. Un stock no permanece tras su salida económica.
G. Las reasignaciones pre-fill no crean exposición en origen.
H. SPY comienza en inception.
I. Drawdown visible.
J. PRICE_RETURN_TWR visible.
K. INCOME_NOT_INCLUDED visible.
L. Confianza parcial visible.
M. Cobertura visible.
N. Selectores funcionan.
O. Tooltip completo.
P. Responsive.
Q. Loading/error/empty.
R. Navegación sin escrituras.
S. Renderer anterior inactivo.
T. Hard cap inactivo.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- Gate 2E-A;
- Gate 2D-2;
- financial core;
- daily-close;
- startup catch-up;
- repository integrity.

==================================================
22. DESPLIEGUE
==================================================

No desplegar hasta:

- cinco carteras visibles;
- cinco curvas correctas;
- SPY visible;
- drawdown visible;
- confianza visible;
- cobertura visible;
- Chrome desktop verde;
- Chrome móvil verde;
- cero errores JavaScript;
- navegación read-only;
- rollback documentado;
- secret scan verde.

Antes:

- backup;
- integrity_check;
- foreign_key_check;
- baseline;
- una sola instancia;
- git limpio salvo cambios autorizados.

Después:

- una sola instancia;
- health;
- env-info;
- cinco curvas visibles;
- hard cap inactivo;
- datos económicos intactos.

==================================================
23. CÓDIGO Y COMMIT
==================================================

Añadir exclusivamente:

- LensPaperTrackView;
- CSS aislado;
- selector de cartera;
- selectores temporales;
- gráfica;
- SPY;
- drawdown;
- KPIs;
- confianza;
- cobertura;
- disclaimer;
- pruebas;
- rollback visual.

No añadir:

- referencias a Crecimiento;
- adaptadores de Crecimiento;
- componentes compartidos;
- hard-cap funcional;
- nuevas asignaciones;
- cambios de schema;
- cambios del motor;
- scratchpad;
- backups;
- bases;
- secretos.

Ejecutar secret scan.

Crear un commit local:

feat(lens): add catalyzed portfolio track record charts

No crear tag.
No hacer push.

==================================================
24. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = 4565e22
CANONICAL_HEAD_AFTER = <hash>

SCHEMA_BEFORE = v29
SCHEMA_AFTER = v29

PAPER_VIEW_COMPONENT =
LensPaperTrackView

PAPER_DATA_SOURCE =
GET /lens/paper-track/:portfolioId

PAPER_PORTFOLIOS_VISIBLE = <número>/5
PAPER_CURVES_VISIBLE = <número>/5
PAPER_SPY_VISIBLE = <número>/5
PAPER_DRAWDOWN_VISIBLE = <número>/5
PAPER_KPIS_VISIBLE = <número>/5

PAPER_MAIN_SERIES =
PAPER_UNIT_VALUE

PAPER_COMPOSITION_POLICY =
POINT_IN_TIME_AS_OPERATED

PRICE_RETURN_TWR_VISIBLE = PASS/FAIL
INCOME_NOT_INCLUDED_VISIBLE = PASS/FAIL
PAPER_SIMULATION_DISCLAIMER_VISIBLE = PASS/FAIL
PARTIAL_HISTORICAL_CONFIDENCE_VISIBLE = PASS/FAIL
PRICE_COVERAGE_VISIBLE = PASS/FAIL
LAST_CERTIFIED_SESSION_VISIBLE = PASS/FAIL

PAPER_CSS_NAMESPACED = PASS/FAIL
PAPER_DOM_ISOLATED = PASS/FAIL

PAPERCHARTHTML_STATUS =
REMOVED /
DEPRECATED_INACTIVE /
ACTIVE_BLOCKER

TEMPORAL_SELECTORS_GREEN = PASS/FAIL
TOOLTIPS_GREEN = PASS/FAIL
LOADING_STATES_GREEN = PASS/FAIL
ERROR_STATES_GREEN = PASS/FAIL
EMPTY_STATES_GREEN = PASS/FAIL
RESPONSIVE_GREEN = PASS/FAIL

CHROME_DESKTOP_1440_GREEN = PASS/FAIL
CHROME_DESKTOP_1280_GREEN = PASS/FAIL
CHROME_MOBILE_390_GREEN = PASS/FAIL
CHROME_MOBILE_360_GREEN = PASS/FAIL

JAVASCRIPT_ERRORS = <número>
UI_NAVIGATION_DATABASE_WRITES = <número>

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

HARD_CAP_ACTIVE =
FALSE

NO_POLICY_ACTIVATED = PASS/FAIL
NO_ENGINE_CHANGE = PASS/FAIL
NO_SCHEMA_CHANGE = PASS/FAIL
NO_REAL_ECONOMIC_WRITES = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

LOCAL_COMMIT = <hash>

Detenerse después de mostrar y validar las cinco gráficas.