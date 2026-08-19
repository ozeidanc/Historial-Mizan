MIZAN · LA LENTE
GATE 2E-B · VISTA DEL TRACK RECORD PAPEL
PARIDAD DE ESTILO CON LA GRÁFICA DE CRECIMIENTO
SIN MODIFICAR CRECIMIENTO

ACLARACIÓN DE ALCANCE

La cartera de Crecimiento y su Track Record no forman parte funcional,
arquitectónica ni económica de La Lente.

La gráfica de Crecimiento se utiliza exclusivamente como REFERENCIA VISUAL.

No se autoriza:

- compartir su componente;
- adaptar su renderer;
- cambiar su código;
- cambiar su DOM;
- cambiar su CSS;
- cambiar sus listeners;
- cambiar sus endpoints;
- cambiar sus datos;
- cambiar su metodología;
- hacer que Crecimiento consuma un componente nuevo.

No crear un TrackRecordView compartido entre Crecimiento y Papel.

La misión es crear una vista aislada para La Lente que reproduzca el estilo
visual de la gráfica de Crecimiento, sin establecer una dependencia funcional
con ella.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = 4565e22
schema esperado = v29
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Gate 2E-A confirmado:

VERSIONED_MANIFEST_EXTENSION = PASS
FUTURE_SESSION_CATCHUP = PASS
FAIL_CLOSED_PRICE_INGESTION = PASS
HISTORICAL_PRICE_IMMUTABILITY = PASS

Estado Papel:

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
1. MISIÓN
==================================================

Crear una vista específica e independiente para el Track Record Papel de La
Lente.

Nombre propuesto:

LensPaperTrackView

o un nombre equivalente claramente limitado a La Lente.

La vista debe:

1. consumir exclusivamente el endpoint canónico Papel;
2. mostrar la serie PRICE_RETURN_TWR;
3. mostrar el benchmark SPY;
4. mostrar drawdown;
5. mostrar KPIs Papel;
6. mostrar cobertura y frescura;
7. mostrar confianza histórica;
8. reproducir el estilo visual de la gráfica de Crecimiento;
9. funcionar en escritorio y móvil;
10. sustituir el renderer Papel divergente;
11. no modificar absolutamente nada de Crecimiento.

==================================================
2. RESTRICCIONES ABSOLUTAS
==================================================

No modificar ningún archivo, bloque, función o selector perteneciente a:

- cartera de Crecimiento;
- Track Record de Crecimiento;
- renderer de Crecimiento;
- endpoint de Crecimiento;
- datos de Crecimiento;
- metodología de Crecimiento;
- navegación de Crecimiento;
- selectores de Crecimiento;
- tooltip de Crecimiento;
- drawdown de Crecimiento;
- KPIs de Crecimiento.

No crear:

- Growth adapter;
- shared Growth/Paper adapter;
- componente compartido que obligue a modificar Crecimiento;
- dependencia de Crecimiento sobre código nuevo de La Lente.

No cambiar:

- Book real;
- cash real;
- NAV real;
- valuations reales;
- holdings legacy;
- snapshots legacy;
- movimientos;
- decisiones;
- resizing;
- tesis;
- catalizadores;
- Campo de Caza;
- daily-close;
- startup catch-up;
- motor TWR;
- snapshots Papel;
- manifiesto de precios;
- endpoints económicos.

No activar:

MANUAL_NOTIONAL_HARD_CAP

No fijar:

effective_from

No hacer push.
No crear tag.
No realizar operaciones Git remotas.

==================================================
3. CRECIMIENTO COMO REFERENCIA VISUAL INMUTABLE
==================================================

Antes de implementar, inspeccionar la gráfica de Crecimiento únicamente para
documentar su contrato visual.

Capturar:

- dimensiones;
- estructura;
- espaciado;
- tipografía;
- colores;
- línea principal;
- línea benchmark;
- ejes;
- grid;
- leyenda;
- selectores temporales;
- tooltip;
- tarjetas KPI;
- drawdown;
- loading;
- error;
- empty state;
- responsive.

Generar un inventario visual:

GROWTH_VISUAL_REFERENCE

con:

- chart height;
- chart margins;
- axis styles;
- primary series style;
- benchmark style;
- tooltip layout;
- KPI layout;
- drawdown layout;
- selector layout;
- responsive breakpoints;
- number formatting;
- date formatting.

La inspección de Crecimiento debe ser read-only.

No extraer código desde Crecimiento si la extracción requiere cambiar su
implementación.

Puede reutilizarse una utilidad ya genérica y estable únicamente si:

- Crecimiento ya la utiliza sin modificación;
- La Lente puede llamarla sin cambiar Crecimiento;
- no se introducen efectos globales;
- no se altera ningún consumidor existente.

Si no existe una utilidad genérica segura:

- implementar LensPaperTrackView de forma aislada;
- reproducir el contrato visual;
- no refactorizar Crecimiento.

==================================================
4. AISLAMIENTO CSS Y DOM
==================================================

La vista Papel debe utilizar un namespace propio.

Ejemplo:

.lens-paper-track
.lens-paper-track__chart
.lens-paper-track__tooltip
.lens-paper-track__kpi
.lens-paper-track__drawdown

No modificar selectores CSS globales que puedan afectar a Crecimiento.

No utilizar selectores genéricos como:

.track-chart
.chart-tooltip
.track-kpi

si cambiar sus reglas puede alterar otra vista.

Si se reutilizan variables visuales existentes:

- solo leerlas;
- no cambiar sus valores globales;
- no redefinirlas para Crecimiento.

Evitar:

- IDs DOM compartidos;
- listeners globales;
- variables globales mutables;
- colisiones de eventos;
- selectores que alcancen ambas vistas.

Entregar:

PAPER_CSS_NAMESPACED = PASS/FAIL
PAPER_DOM_ISOLATED = PASS/FAIL
GROWTH_CSS_TOUCHED = <número de reglas>
GROWTH_DOM_TOUCHED = <número de elementos>

Resultados obligatorios:

GROWTH_CSS_TOUCHED = 0
GROWTH_DOM_TOUCHED = 0

==================================================
5. FUENTE ECONÓMICA DE PAPEL
==================================================

LensPaperTrackView debe consumir exclusivamente:

GET /lens/paper-track/:portfolioId

Mapeo:

- paper_unit_value → serie principal;
- cumulative_return → retorno acumulado;
- daily_return → retorno diario;
- paper_nav → Paper NAV;
- paper_cash → Paper Cash;
- paper_market_value → exposición;
- benchmark_unit_value → serie SPY;
- benchmark_cumulative_return → retorno SPY;
- drawdown → drawdown;
- max_drawdown → máximo drawdown;
- historical_confidence → confianza;
- coverage → cobertura;
- last_session → última sesión certificada.

No utilizar:

- holdings actuales;
- snapshots legacy;
- precio_entrada;
- coste medio;
- endpoint de Crecimiento;
- payload de Crecimiento;
- renderer de Crecimiento como fuente de datos;
- cálculos económicos frontend.

La UI no puede recalcular:

- NAV;
- TWR;
- SPY;
- drawdown;
- cobertura;
- confianza.

==================================================
6. CONTRATO DE LENSPAPERTRACKVIEW
==================================================

LensPaperTrackView debe recibir un read model de presentación:

- portfolio_id;
- title;
- portfolio_kind = PAPER;
- methodology;
- income_policy;
- benchmark;
- first_session;
- last_session;
- as_of;
- latest_closed_session;
- series;
- benchmark_series;
- metrics;
- windows;
- drawdown;
- coverage;
- freshness;
- historical_confidence;
- warnings;
- disclaimer;
- loading_state;
- error_state;
- empty_state.

La vista debe limitarse a:

- seleccionar ventanas;
- renderizar;
- formatear;
- mostrar estados;
- gestionar tooltips;
- gestionar responsive.

No debe realizar operaciones económicas ni de persistencia.

==================================================
7. PARIDAD DE ESTILO
==================================================

La gráfica Papel debe reproducir el estilo visual observado en Crecimiento:

- mismo tipo de contenedor;
- misma jerarquía visual;
- misma altura de gráfico;
- mismos márgenes visuales;
- mismos ejes;
- mismo grid;
- mismas convenciones de color;
- mismo estilo de serie principal;
- mismo estilo de benchmark;
- misma disposición de leyenda;
- mismo selector temporal;
- misma estructura de tooltip;
- misma presentación de drawdown;
- misma disposición de KPIs;
- mismos formatos de fechas;
- mismos formatos de porcentajes;
- mismos formatos monetarios;
- misma lógica responsive;
- mismos estados de carga y error.

Esto es paridad de estilo.

No significa:

- componente compartido;
- renderer compartido;
- datos compartidos;
- metodología compartida;
- endpoint compartido;
- modificación de Crecimiento.

==================================================
8. DIFERENCIAS OBLIGATORIAS DE PAPEL
==================================================

La vista debe mostrar claramente:

- PAPEL;
- nombre de la cartera;
- PRICE_RETURN_TWR;
- Paper NAV;
- Paper Unit Value;
- benchmark SPY;
- INCOME_NOT_INCLUDED;
- simulación;
- primera sesión;
- última sesión certificada;
- sesiones valoradas;
- posiciones económicas activas;
- paper cash;
- exposición;
- cobertura;
- confianza histórica.

Disclaimer:

“Simulación papel point-in-time. Rentabilidad por precio; dividendos no
incluidos.”

No denominar:

- rentabilidad real;
- ejecución real;
- total return;
- Track de Crecimiento;
- alpha estadística.

==================================================
9. CONFIANZA HISTÓRICA
==================================================

Mostrar:

PARTIAL_HISTORICAL_CONFIDENCE

cuando la ventana seleccionada contenga sesiones parciales.

Texto conceptual:

“Parte del historial fue reconstruida con composición y precios point-in-time,
pero el importe solicitado original de algunas posiciones no está
disponible.”

Mostrar:

- sesiones parciales de la ventana;
- objetos sujetos a revisión;
- motivo;
- estado MANUAL_REVIEW_REQUIRED.

No mostrar una advertencia parcial si la ventana seleccionada contiene solo
sesiones FULL_HISTORICAL_CONFIDENCE.

No presentar los 36 objetos inferidos como exactos.

==================================================
10. FRESCURA Y COBERTURA
==================================================

Mostrar:

- última sesión NYSE cerrada;
- última sesión certificada;
- as_of;
- coverage status;
- catch-up status;
- price basis.

Estados:

- CURRENT;
- CATCHUP_PENDING;
- PRICE_COVERAGE_INCOMPLETE;
- PARTIAL_HISTORICAL_CONFIDENCE;
- BLOCKED.

Si:

latest_closed_session > last_certified_session

mostrar:

CATCHUP_PENDING

o:

PRICE_COVERAGE_INCOMPLETE

No presentar la curva como actual.

No utilizar datos live para completar la curva.

==================================================
11. SELECTORES TEMPORALES
==================================================

Reproducir los selectores visuales de la gráfica de Crecimiento.

Usar las mismas etiquetas disponibles:

- 1D;
- MTD;
- YTD;
- 1Y;

o la lista exacta observada en la referencia visual.

La implementación pertenece exclusivamente a La Lente.

No reutilizar handlers globales de Crecimiento si pueden crear acoplamiento o
efectos colaterales.

Para historia corta:

- recortar a inception;
- no inventar puntos;
- conservar selector activo;
- mostrar historial disponible desde la fecha real.

==================================================
12. TOOLTIP PAPEL
==================================================

Mostrar:

- sesión;
- Paper Unit Value;
- retorno diario;
- retorno acumulado;
- Paper NAV;
- Paper Cash;
- exposición;
- SPY;
- diferencial simple frente a SPY;
- drawdown;
- confianza;
- cobertura.

No mostrar desconocidos como cero.

Usar:

- No disponible;
- warning;
- estado explícito.

No denominar el diferencial “alpha”.

==================================================
13. KPIS PAPEL
==================================================

Mostrar:

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

No añadir métricas no soportadas por el endpoint.

==================================================
14. DRAWDOWN
==================================================

Mostrar:

- drawdown actual;
- máximo drawdown;
- sección o gráfica de drawdown con el estilo visual de la referencia.

No utilizar:

mdd = 0

como fallback.

Si falta:

- NULL;
- No disponible;
- estado explícito.

Convención:

drawdown <= 0
max_drawdown <= 0

==================================================
15. ESTADOS DE UI
==================================================

Validar:

- loading;
- timeout;
- cartera inexistente;
- serie vacía;
- endpoint 4xx;
- endpoint 5xx;
- catch-up pendiente;
- cobertura incompleta;
- confianza parcial;
- blocked;
- última sesión válida disponible.

La UI no debe:

- quedar en blanco;
- mostrar ceros falsos;
- obtener precios;
- crear snapshots;
- ejecutar catch-up;
- activar políticas;
- escribir en la base.

==================================================
16. PAPELCHARTHTML
==================================================

No eliminar papelChartHTML al comenzar.

Proceso:

1. inventariar consumidores;
2. implementar LensPaperTrackView;
3. conectar el endpoint Papel;
4. validar las cinco carteras;
5. validar Chrome;
6. validar responsive;
7. verificar listeners;
8. verificar navegación;
9. poner LensPaperTrackView como renderer activo;
10. dejar papelChartHTML inactivo;
11. eliminarlo solo si tiene cero consumidores.

No mantener dos renderers activos.

Estados permitidos al final:

REMOVED

o:

DEPRECATED_INACTIVE

No aceptar:

ACTIVE

==================================================
17. CRECIMIENTO: VERIFICACIÓN SOLO OBSERVACIONAL
==================================================

Crecimiento no se modifica.

La validación consiste exclusivamente en comprobar que sigue igual después
de desplegar cambios Papel.

Comparar:

- archivos relacionados;
- hash de código;
- payload;
- serie;
- métricas;
- benchmark;
- drawdown;
- DOM;
- capturas;
- comportamiento.

Resultados obligatorios:

GROWTH_CODE_UNCHANGED = PASS
GROWTH_DATA_UNCHANGED = PASS
GROWTH_DOM_UNCHANGED = PASS
GROWTH_CSS_UNCHANGED = PASS
GROWTH_BEHAVIOR_UNCHANGED = PASS

No crear adaptadores para Crecimiento.

No hacer que Crecimiento use LensPaperTrackView.

==================================================
18. CHROME
==================================================

Validar en Chrome real.

Papel:

- cinco carteras;
- escritorio 1440 × 900;
- escritorio 1280 × 720;
- móvil 390 × 844;
- móvil 360 × 800;
- todos los selectores;
- tooltip;
- SPY;
- drawdown;
- KPIs;
- confianza;
- cobertura;
- disclaimer;
- loading;
- error;
- empty.

Verificar:

- cero overflow horizontal;
- cero clipping;
- cero superposición;
- ejes legibles;
- tooltip dentro del viewport;
- benchmark visible;
- drawdown visible;
- cero errores JavaScript.

Crecimiento:

- abrir antes y después únicamente para comprobar que permanece intacto.

==================================================
19. COMPARACIÓN VISUAL
==================================================

Comparar:

A. REFERENCIA VISUAL DE CRECIMIENTO

Se utiliza para medir estilo y estructura.

B. RESULTADO PAPEL

Debe reproducir:

- layout;
- gráfico;
- ejes;
- selector;
- tooltip;
- benchmark;
- drawdown;
- KPIs;
- responsive.

No comparar valores, textos o metodología como si debieran coincidir.

La comparación visual debe permitir diferencias legítimas:

- nombre de cartera;
- etiqueta PAPEL;
- valores;
- fechas;
- warnings;
- disclaimer;
- confianza;
- cobertura;
- antialiasing.

Declarar:

PAPER_STYLE_PARITY_WITH_GROWTH_REFERENCE = PASS/FAIL

No declarar:

GROWTH_SHARED_COMPONENT

==================================================
20. UI READ-ONLY
==================================================

Capturar contadores antes y después de:

- abrir las cinco carteras;
- cambiar ventanas;
- usar tooltips;
- recargar;
- cambiar de cartera;
- visitar Crecimiento.

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
21. ROLLBACK VISUAL
==================================================

Preparar rollback únicamente del frontend Papel.

Debe restaurar:

- renderer Papel anterior;
- navegación Papel anterior.

No debe revertir:

- motor TWR;
- snapshots;
- manifiesto;
- daily-close;
- endpoints;
- schema;
- datos económicos.

Documentar:

- commit anterior;
- archivos afectados;
- procedimiento;
- validación posterior.

==================================================
22. PRUEBAS
==================================================

Crear:

verify-lens-paper-track-view-contract.mjs
verify-lens-paper-track-view-adapter.mjs
verify-lens-paper-track-style-parity.mjs
verify-lens-paper-track-confidence-ui.mjs
verify-lens-paper-track-coverage-ui.mjs
verify-lens-paper-track-readonly-ui.mjs
verify-lens-paper-track-responsive.mjs
verify-lens-paper-track-chrome.mjs
verify-growth-untouched-by-lens-ui.mjs

Casos:

A. Papel consume endpoint canónico.
B. Papel no consume endpoint de Crecimiento.
C. Papel no calcula TWR.
D. Papel muestra SPY.
E. Papel muestra drawdown.
F. Papel muestra PRICE_RETURN_TWR.
G. Papel muestra INCOME_NOT_INCLUDED.
H. Confianza parcial visible por ventana.
I. Cobertura y frescura visibles.
J. Selectores funcionan.
K. Tooltip completo.
L. Responsive.
M. Estados loading/error/empty.
N. Navegación = cero escrituras.
O. papelChartHTML inactivo.
P. Crecimiento code unchanged.
Q. Crecimiento DOM unchanged.
R. Crecimiento CSS unchanged.
S. Crecimiento data unchanged.
T. Hard cap inactivo.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- suites Gate 2E-A;
- suites Gate 2D-2;
- financial core;
- daily-close;
- startup catch-up;
- repository integrity.

==================================================
23. DESPLIEGUE
==================================================

No desplegar hasta:

- las cinco carteras Papel están verdes;
- Chrome desktop y móvil están verdes;
- visual parity está verde;
- Crecimiento está intacto;
- navegación read-only;
- cero errores JavaScript;
- rollback documentado;
- secret scan verde.

Antes:

- backup;
- integrity_check;
- foreign_key_check;
- baseline;
- capturas;
- una sola instancia;
- git limpio salvo cambios autorizados.

Después:

- una sola instancia;
- health;
- env-info;
- cinco curvas Papel visibles;
- Crecimiento intacto;
- hard cap inactivo;
- datos económicos intactos.

==================================================
24. CÓDIGO Y COMMIT
==================================================

Añadir exclusivamente:

- LensPaperTrackView;
- adaptador Papel;
- CSS Papel aislado;
- confianza;
- cobertura;
- disclaimer;
- pruebas;
- documentación de rollback.

No añadir:

- adaptador de Crecimiento;
- componente compartido con Crecimiento;
- modificaciones de Crecimiento;
- hard-cap funcional;
- nuevas asignaciones;
- schema;
- motor económico;
- scratchpad;
- backups;
- bases;
- secretos.

Ejecutar secret scan.

Crear un commit local:

feat(lens): add growth-style paper track view

No crear tag.
No hacer push.

==================================================
25. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = 4565e22
CANONICAL_HEAD_AFTER = <hash>

SCHEMA_BEFORE = v29
SCHEMA_AFTER = v29

PAPER_VIEW_COMPONENT =
LensPaperTrackView

PAPER_DATA_SOURCE =
GET /lens/paper-track/:portfolioId

PAPER_STYLE_REFERENCE =
GROWTH_TRACK_VISUAL_STYLE_ONLY

SHARED_COMPONENT_WITH_GROWTH =
FALSE

GROWTH_CODE_UNCHANGED = PASS/FAIL
GROWTH_DATA_UNCHANGED = PASS/FAIL
GROWTH_DOM_UNCHANGED = PASS/FAIL
GROWTH_CSS_UNCHANGED = PASS/FAIL
GROWTH_BEHAVIOR_UNCHANGED = PASS/FAIL

PAPER_CSS_NAMESPACED = PASS/FAIL
PAPER_DOM_ISOLATED = PASS/FAIL

PAPERCHARTHTML_CONSUMERS_BEFORE = <número>
PAPERCHARTHTML_CONSUMERS_AFTER = <número>

PAPERCHARTHTML_STATUS =
REMOVED /
DEPRECATED_INACTIVE /
ACTIVE_BLOCKER

PAPER_PORTFOLIOS_VISIBLE = <número>/5
PAPER_CURVES_VISIBLE = <número>/5
PAPER_SPY_VISIBLE = <número>/5
PAPER_DRAWDOWN_VISIBLE = <número>/5
PAPER_KPIS_VISIBLE = <número>/5

PAPER_STYLE_PARITY_WITH_GROWTH_REFERENCE = PASS/FAIL

PRICE_RETURN_TWR_VISIBLE = PASS/FAIL
INCOME_NOT_INCLUDED_VISIBLE = PASS/FAIL
PAPER_SIMULATION_DISCLAIMER_VISIBLE = PASS/FAIL
PARTIAL_HISTORICAL_CONFIDENCE_VISIBLE = PASS/FAIL
PRICE_COVERAGE_VISIBLE = PASS/FAIL
LAST_CERTIFIED_SESSION_VISIBLE = PASS/FAIL

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

VISUAL_ROLLBACK_DOCUMENTED = PASS/FAIL
SECRET_SCAN = PASS/FAIL

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

EXACT_BLOCKERS_BEFORE_GATE_2F:
1. <bloqueo>
2. <bloqueo>
3. <bloqueo>

Detenerse después del Gate 2E-B.