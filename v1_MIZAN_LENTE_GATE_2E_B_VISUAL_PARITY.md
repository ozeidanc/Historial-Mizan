MIZAN · LA LENTE
GATE 2E-B · PARIDAD VISUAL DEL TRACK RECORD PAPEL
TRACKRECORDVIEW COMPARTIDO · ADAPTADORES · CHROME · CERO REGRESIÓN

NATURALEZA

Esta ejecución autoriza exclusivamente:

1. capturar la línea base visual y funcional de Crecimiento;
2. capturar la línea base actual de Papel;
3. crear o consolidar TrackRecordView;
4. adaptar Crecimiento al componente compartido sin cambiar sus datos ni
   comportamiento;
5. adaptar Papel al componente compartido usando exclusivamente el endpoint
   canónico;
6. mostrar SPY, drawdown, cobertura y confianza;
7. validar escritorio, móvil, ventanas y tooltips en Chrome;
8. desplegar la nueva vista únicamente después de todas las validaciones;
9. dejar el renderer anterior de Papel fuera de uso;
10. eliminarlo solo si se demuestra que no tiene consumidores;
11. crear un único commit local.

No autoriza:

- activar MANUAL_NOTIONAL_HARD_CAP;
- fijar effective_from;
- cambiar incorporaciones papel;
- crear solicitudes o asignaciones;
- modificar el motor PRICE_RETURN_TWR;
- modificar snapshots papel;
- modificar la ingesta de precios ya validada;
- modificar daily-close;
- modificar startup catch-up;
- modificar endpoints económicos salvo ajustes de compatibilidad estrictamente
  read-only;
- modificar la metodología de Crecimiento;
- modificar posiciones reales;
- modificar Book, cash o NAV reales;
- modificar holdings o snapshots legacy;
- modificar tesis;
- modificar catalizadores;
- modificar Campo de Caza;
- crear tag;
- hacer push;
- realizar operaciones Git remotas.

Gate 2E-B es exclusivamente frontend/read-model.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = 4565e22
schema esperado = v29
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Gate 2E-A confirmado:

VERSIONED_MANIFEST_EXTENSION =
PASS

FUTURE_SESSION_CATCHUP =
PASS

FAIL_CLOSED_PRICE_INGESTION =
PASS

HISTORICAL_PRICE_IMMUTABILITY =
PASS

CURRENT_MANIFEST_VERSION =
1

CURRENT_COVERAGE_END_SESSION =
2026-07-24

Estado Track Papel:

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

Endpoint principal:

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
1. PRECONDICIONES
==================================================

Antes de tocar el dashboard:

- confirmar HEAD 4565e22;
- confirmar schema v29;
- confirmar env-info;
- confirmar una sola raíz Git;
- inventariar archivos untracked;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- confirmar una sola instancia;
- confirmar health;
- confirmar endpoints Papel verdes;
- confirmar GET read-only;
- confirmar las 54 filas de paper_track_snapshot;
- confirmar manifiesto y catch-up operativos;
- confirmar hard cap sin effective_from.

Capturar baseline económico de Crecimiento:

- payload completo;
- serie normalizada;
- benchmark;
- métricas;
- ventanas;
- drawdown;
- cobertura;
- primera y última sesión;
- hashes económicos.

Capturar baseline visual de Crecimiento en Chrome:

- desktop;
- móvil;
- selector 1D;
- selector MTD;
- selector YTD;
- selector 1Y;
- tooltip;
- drawdown;
- benchmark;
- loading;
- error;
- cobertura.

Capturar baseline de las cinco carteras Papel:

- renderer actual;
- series;
- fechas;
- benchmark;
- KPIs;
- warnings;
- escritorio;
- móvil.

No continuar si:

- el endpoint Papel no coincide con los snapshots persistidos;
- existe una sesión bloqueada;
- Crecimiento no está verde antes del refactor;
- el hard cap aparece activo;
- integrity_check o foreign_key_check falla.

==================================================
2. INVENTARIO DEL RENDERER ACTUAL
==================================================

Localizar e inventariar:

- renderTrack;
- renderPapelTrack;
- trkGraficaSVG;
- papelChartHTML;
- papelChartHTML consumers;
- selectores temporales;
- handlers de tooltip;
- cálculo o render de drawdown;
- tarjetas KPI;
- estados loading/error/empty;
- CSS específico;
- listeners;
- rutas de navegación;
- IDs DOM;
- referencias globales.

Entregar antes del cambio:

GROWTH_RENDER_PATH =
<funciones>

PAPER_RENDER_PATH =
<funciones>

SHARED_RENDER_FUNCTIONS =
<lista>

PAPERCHARTHTML_CONSUMERS =
<número y lista>

DOM_ID_COLLISIONS =
<número>

GLOBAL_LISTENER_RISKS =
<número>

No eliminar código antes de completar este inventario.

==================================================
3. CONTRATO NORMALIZADO
==================================================

Crear un contrato único para TrackRecordView.

Campos mínimos:

- portfolio_id;
- portfolio_kind;
- title;
- subtitle;
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
- drawdown;
- coverage;
- freshness;
- historical_confidence;
- warnings;
- disclaimer;
- loading_state;
- error_state;
- empty_state;
- labels;
- formatting.

TrackRecordView no debe:

- consultar tablas;
- obtener precios;
- calcular NAV;
- recalcular TWR;
- recalcular SPY;
- recalcular confianza;
- crear snapshots;
- modificar estado.

Debe limitarse a:

- normalizar presentación;
- seleccionar ventanas;
- renderizar;
- formatear;
- gestionar interacción visual.

==================================================
4. ADAPTADOR DE CRECIMIENTO
==================================================

Crear un adaptador de presentación para el payload actual de Crecimiento.

No modificar:

- endpoint;
- posicionPnL;
- valorEnFecha;
- valuations;
- metodología de retorno;
- benchmark;
- métricas;
- ventanas;
- riesgo;
- drawdown;
- datos históricos.

El adaptador debe preservar:

- valores exactos;
- orden de puntos;
- fechas;
- escalas;
- etiquetas;
- selectores;
- tooltip;
- colores;
- responsive;
- mensajes existentes.

Comparación obligatoria antes/después:

- series;
- benchmark;
- KPIs;
- drawdown;
- ventanas;
- cobertura;
- DOM semántico;
- interacción.

Resultado obligatorio:

GROWTH_TRACK_DATA_UNCHANGED =
PASS

GROWTH_TRACK_BEHAVIOR_UNCHANGED =
PASS

No aceptar una regresión bajo el argumento de refactor.

==================================================
5. ADAPTADOR DE PAPEL
==================================================

Papel debe consumir exclusivamente:

GET /lens/paper-track/:portfolioId

Mapeo principal:

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
- price coverage → cobertura;
- last_session → última sesión certificada.

No usar como fuente económica:

- holdings actuales;
- snapshots legacy;
- precio_entrada;
- coste medio;
- endpoint Track Papel anterior;
- papelChartHTML;
- cálculos frontend.

La UI no debe reconstruir la metodología.

==================================================
6. TRACKRECORDVIEW COMPARTIDO
==================================================

Crear o consolidar:

TrackRecordView

Debe compartir entre Crecimiento y Papel:

- contenedor;
- cabecera;
- gráfico;
- SVG o renderer;
- ejes;
- escalas;
- grid;
- leyenda;
- selector temporal;
- tooltips;
- benchmark;
- drawdown;
- KPIs;
- cobertura;
- loading;
- error;
- empty;
- responsive;
- tipografía;
- formatos numéricos;
- formatos temporales;
- accesibilidad.

La implementación puede ser:

- función compartida;
- módulo;
- componente;

según la arquitectura actual.

No introducir un framework nuevo.

No duplicar dos versiones de TrackRecordView.

==================================================
7. PARIDAD VISUAL
==================================================

Papel debe usar la misma estructura visual que Crecimiento.

Paridad obligatoria:

- misma jerarquía visual;
- mismo ancho y espaciado;
- misma altura del gráfico;
- mismos ejes;
- mismos selectores;
- mismas convenciones de color;
- mismo estilo de benchmark;
- mismo estilo de drawdown;
- mismas tarjetas KPI;
- mismo tooltip;
- mismos estados;
- mismo comportamiento responsive.

“Visualmente idéntico” significa:

- mismo sistema visual;
- mismo layout;
- mismo comportamiento;
- mismos componentes.

No exige una comparación ciega de píxeles cuando existan diferencias
legítimas de:

- texto;
- longitud del nombre;
- valores;
- antialiasing;
- timestamp;
- etiqueta PAPEL;
- warnings;
- disclaimer.

La comparación visual automatizada debe usar:

- regiones estables;
- tolerancia documentada;
- exclusión de timestamps dinámicos;
- dimensiones idénticas de viewport.

==================================================
8. DIFERENCIAS OBLIGATORIAS DE PAPEL
==================================================

Mostrar claramente:

- PAPEL;
- nombre de cartera;
- PRICE_RETURN_TWR;
- Paper NAV;
- Paper Unit Value;
- SPY;
- INCOME_NOT_INCLUDED;
- simulación;
- primera sesión;
- última sesión certificada;
- sesiones valoradas;
- posiciones activas;
- paper cash;
- exposición;
- cobertura;
- confianza histórica.

No denominar:

- retorno real;
- ejecución real;
- total return;
- rentabilidad de Crecimiento;
- alpha estadística.

Disclaimer mínimo conceptual:

“Simulación papel point-in-time. Rentabilidad por precio; dividendos no
incluidos.”

==================================================
9. CONFIANZA HISTÓRICA
==================================================

Cuando la ventana seleccionada contenga alguna sesión parcial, mostrar:

PARTIAL_HISTORICAL_CONFIDENCE

Mensaje conceptual:

“Parte del historial fue reconstruida con composición y precios point-in-time,
pero el importe solicitado original de algunas posiciones no está
disponible.”

Mostrar, cuando esté disponible:

- sesiones parciales dentro de la ventana;
- objetos sujetos a revisión;
- motivo;
- estado MANUAL_REVIEW_REQUIRED.

No mostrar advertencia global si la ventana seleccionada solo contiene
sesiones FULL_HISTORICAL_CONFIDENCE.

La confianza debe recalcularse para la ventana visual usando los estados ya
entregados por el endpoint, no reinterpretando los datos económicos.

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

Estados mínimos:

- CURRENT;
- CATCHUP_PENDING;
- PRICE_COVERAGE_INCOMPLETE;
- PARTIAL_HISTORICAL_CONFIDENCE;
- BLOCKED.

Si:

latest_closed_session > last_certified_session

no presentar la serie como actual.

Mostrar:

CATCHUP_PENDING

o:

PRICE_COVERAGE_INCOMPLETE

según el endpoint.

No usar precio live para completar visualmente la curva.

==================================================
11. SELECTORES TEMPORALES
==================================================

Usar exactamente las ventanas existentes en Crecimiento:

- 1D;
- MTD;
- YTD;
- 1Y.

Si existe otra lista canónica en el código, reutilizarla sin duplicarla.

Para una cartera con historia más corta:

- recortar a inception;
- no inventar puntos;
- mostrar fecha inicial real;
- conservar selector activo;
- explicar “historial disponible desde <fecha>” si procede.

Validar:

- cambio de ventana;
- redimensionado;
- tooltip después del cambio;
- benchmark;
- drawdown;
- confianza por ventana;
- cobertura por ventana.

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

No mostrar valores desconocidos como cero.

Utilizar:

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
- último retorno diario;
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

No añadir métricas que el endpoint no soporte formalmente.

==================================================
14. DRAWNDOWN
==================================================

Papel debe mostrar:

- curva o sección de drawdown equivalente a Crecimiento;
- drawdown actual;
- máximo drawdown.

No utilizar:

mdd = 0

como fallback.

Si no está disponible:

- NULL;
- No disponible;
- estado explícito.

Validar convención:

- drawdown <= 0;
- máximo drawdown <= 0.

==================================================
15. ESTADOS DE CARGA, ERROR Y VACÍO
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
- disparar catch-up;
- modificar estado.

==================================================
16. PAPELCHARTHTML
==================================================

No eliminar papelChartHTML al comenzar.

Proceso:

1. implementar TrackRecordView;
2. adaptar Crecimiento;
3. validar Crecimiento;
4. adaptar Papel;
5. validar Papel;
6. verificar búsquedas de consumidores;
7. comprobar listeners;
8. comprobar navegación;
9. desactivar papelChartHTML;
10. mantener fallback temporal durante validación;
11. eliminar únicamente si cero consumidores y rollback preparado.

Resultado esperado:

PAPER_TRACK_RENDERER =
TrackRecordView

Si papelChartHTML debe permanecer temporalmente:

- marcar deprecated;
- garantizar que no está activo;
- documentar retirada posterior.

No mantener dos renderers activos simultáneamente.

==================================================
17. ROLLBACK VISUAL
==================================================

Preparar rollback inmediato del frontend.

El rollback debe:

- restaurar el renderer anterior;
- no revertir snapshots;
- no revertir motor;
- no revertir manifiesto;
- no modificar schema;
- no modificar datos económicos.

Antes de desplegar identificar:

- commit anterior;
- archivos afectados;
- procedimiento de restauración;
- validación posterior.

Si existe un feature flag local compatible con la arquitectura actual, puede
usarse para seleccionar renderer durante validación.

No introducir infraestructura compleja únicamente para este gate.

==================================================
18. CHROME
==================================================

Validar en Chrome real.

Viewports mínimos:

DESKTOP:
- 1440 × 900;
- 1280 × 720.

MOBILE:
- 390 × 844;
- 360 × 800.

CRECIMIENTO:

- vista inicial;
- 1D;
- MTD;
- YTD;
- 1Y;
- tooltip;
- benchmark;
- drawdown;
- loading/error;
- responsive.

PAPEL:

- cinco carteras;
- cuatro ventanas;
- tooltip;
- benchmark;
- drawdown;
- confianza;
- cobertura;
- disclaimer;
- desktop;
- mobile.

Capturar:

- baseline antes;
- resultado después;
- diferencias.

No aceptar:

- overflow horizontal;
- clipping;
- texto superpuesto;
- ejes ilegibles;
- selectores rotos;
- tooltips fuera de viewport;
- drawdown omitido;
- benchmark ausente;
- JavaScript errors.

==================================================
19. COMPARACIÓN VISUAL
==================================================

Comparar Crecimiento antes/después mediante:

A. DATOS

Igualdad económica exacta.

B. ESTRUCTURA

- mismos bloques;
- mismo orden;
- mismas interacciones.

C. CAPTURAS

Usar una tolerancia documentada para:

- antialiasing;
- fuentes;
- timestamps dinámicos.

No aceptar diferencias materiales de:

- posición;
- dimensiones;
- espaciado;
- escalas;
- colores;
- texto funcional;
- tooltip;
- drawdown;
- benchmark.

Entregar lista exacta de diferencias aceptadas.

==================================================
20. UI READ-ONLY
==================================================

Antes y después de navegar:

- capturar contadores DB;
- abrir las cinco carteras;
- cambiar ventanas;
- activar tooltips;
- recargar;
- cambiar entre Crecimiento y Papel.

Verificar:

UI_NAVIGATION_DATABASE_WRITES = 0

La navegación no puede provocar:

- INSERT;
- UPDATE;
- DELETE;
- price fetch;
- snapshot generation;
- catch-up;
- policy activation.

==================================================
21. REGRESIÓN E INTEGRIDAD
==================================================

Ejecutar:

- suites v27/v28/v29;
- Gate 2C-2;
- Gate 2D-1;
- Gate 2D-2;
- Gate 2E-A;
- financial core;
- Growth Track;
- daily-close;
- startup catch-up;
- repository integrity.

Comparar antes/después:

- payload económico de Crecimiento;
- holdings;
- snapshots;
- valuations;
- movimientos;
- Book;
- cash;
- NAV;
- tesis;
- catalizadores;
- Campo de Caza.

Resultados obligatorios:

GROWTH_TRACK_DATA_UNCHANGED = PASS
REAL_BOOK_UNCHANGED = PASS
REAL_CASH_UNCHANGED = PASS
REAL_NAV_UNCHANGED = PASS
REAL_VALUATIONS_UNCHANGED = PASS
HOLDINGS_LEGACY_UNCHANGED = PASS
SNAPSHOTS_LEGACY_UNCHANGED = PASS
THESES_UNCHANGED = PASS
CATALYSTS_UNCHANGED = PASS
HUNTING_FIELD_UNCHANGED = PASS

==================================================
22. GATE DE DESPLIEGUE
==================================================

No desplegar hasta que:

- pruebas unitarias verdes;
- pruebas de integración verdes;
- Chrome verde;
- capturas revisadas;
- Crecimiento sin regresión;
- cinco carteras Papel visibles;
- cero errores JavaScript;
- navegación read-only;
- rollback documentado;
- secret scan verde.

Antes:

- backup;
- SHA-256;
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
- Chrome;
- TrackRecordView activo;
- cinco curvas Papel visibles;
- Crecimiento intacto;
- hard cap inactivo;
- contadores económicos intactos.

==================================================
23. CÓDIGO Y COMMIT
==================================================

Añadir exclusivamente:

- TrackRecordView;
- adaptador de Crecimiento;
- adaptador de Papel;
- estilos compartidos;
- confianza;
- cobertura;
- disclaimers;
- pruebas UI;
- documentación de rollback.

No añadir:

- hard-cap funcional;
- UI de capacidad;
- nuevas asignaciones;
- cambios de schema;
- cambios del motor económico;
- scratchpad;
- screenshots productivos al repositorio, salvo patrón existente y
  autorización;
- backups;
- bases;
- secretos.

Ejecutar secret scan.

Crear un único commit local:

feat(lens): add shared TrackRecordView for paper portfolios

No crear tag.
No hacer push.

==================================================
24. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = 4565e22
CANONICAL_HEAD_AFTER = <hash>

SCHEMA_BEFORE = v29
SCHEMA_AFTER = v29

GROWTH_RENDER_PATH_BEFORE = <ruta>
GROWTH_RENDER_PATH_AFTER = <ruta>

PAPER_RENDER_PATH_BEFORE = <ruta>
PAPER_RENDER_PATH_AFTER = <ruta>

TRACK_RECORD_VIEW_SHARED = PASS/FAIL

PAPER_TRACK_RENDERER =
TrackRecordView

PAPERCHARTHTML_CONSUMERS_BEFORE = <número>
PAPERCHARTHTML_CONSUMERS_AFTER = <número>
PAPERCHARTHTML_STATUS =
REMOVED /
DEPRECATED_INACTIVE /
ACTIVE_BLOCKER

GROWTH_TRACK_DATA_UNCHANGED = PASS/FAIL
GROWTH_TRACK_BEHAVIOR_UNCHANGED = PASS/FAIL
GROWTH_VISUAL_MATERIAL_DIFFS = <número>
GROWTH_ACCEPTED_VISUAL_DIFFS = <lista>

PAPER_PORTFOLIOS_VISIBLE = <número>/5
PAPER_CURVES_VISIBLE = <número>/5
PAPER_SPY_VISIBLE = <número>/5
PAPER_DRAWDOWN_VISIBLE = <número>/5
PAPER_KPIS_VISIBLE = <número>/5

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
ACCESSIBILITY_BASIC = PASS/FAIL

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