MIZAN · LA LENTE
GATE 2E-B · CIERRE RESPONSIVE Y VALIDACIÓN FINAL

BASELINE

CANONICAL_HEAD = 0466595
schema = v29

La implementación LensPaperTrackView queda aceptada funcionalmente.

No modificar:

- motor económico;
- snapshots;
- endpoints;
- manifiesto;
- daily-close;
- startup catch-up;
- schema;
- política prospectiva;
- renderer, salvo correcciones visuales estrictamente necesarias;
- datos legacy;
- datos reales;
- tesis;
- catalizadores;
- Campo de Caza.

No activar MANUAL_NOTIONAL_HARD_CAP.
No crear tag.
No hacer push.

MISIÓN

Cerrar exclusivamente la validación visual pendiente de LensPaperTrackView.

==================================================
1. CHROME REAL
==================================================

Validar en Chrome real:

DESKTOP

- 1440 × 900;
- 1280 × 720.

MOBILE

- 390 × 844;
- 360 × 800.

En los cuatro viewports validar las cinco carteras:

- cat:catalizada;
- cat:catalizada-2;
- cat:catalizada-2b;
- cat:catalizada-3;
- cat:catalizada-3b.

==================================================
2. INTERACCIONES
==================================================

Validar:

- selector de cartera;
- 1D;
- MTD;
- YTD;
- 1Y;
- MAX;
- línea Paper Unit Value;
- línea SPY;
- KPIs;
- drawdown;
- confianza;
- cobertura;
- disclaimer;
- loading;
- error;
- empty state.

En móvil validar específicamente:

- interacción táctil del tooltip;
- cierre o desplazamiento del tooltip;
- tooltip dentro del viewport;
- selector accesible;
- cambio de cartera;
- cambio de ventana;
- scroll vertical;
- orientación visual correcta.

==================================================
3. CRITERIOS
==================================================

No aceptar:

- overflow horizontal;
- clipping;
- texto superpuesto;
- KPIs cortados;
- ejes ilegibles;
- tooltip fuera del viewport;
- tooltip imposible de cerrar;
- benchmark ausente;
- drawdown omitido;
- selector inaccesible;
- JavaScript errors;
- ceros utilizados como fallback;
- escritura en base de datos.

La historia corta debe:

- recortarse a inception;
- no fabricar puntos;
- mantener la ventana seleccionada;
- mostrar la fecha disponible cuando corresponda.

==================================================
4. RENDERER ANTERIOR
==================================================

Confirmar expresamente:

PAPERCHARTHTML_REMOVED = PASS/FAIL

RENDERPAPELTRACK_STATUS =
COMPATIBILITY_ALIAS_TO_LENSPAPERTRACKVIEW /
REMOVED /
ACTIVE_LEGACY_BLOCKER

Si renderPapelTrack sigue existiendo como alias:

- debe llamar exclusivamente a LensPaperTrackView;
- no debe conservar lógica económica o gráfica antigua;
- no debe crear un segundo renderer activo.

==================================================
5. READ-ONLY
==================================================

Capturar contadores antes y después de toda la navegación Chrome.

Confirmar:

UI_NAVIGATION_DATABASE_WRITES = 0
UI_NAVIGATION_PRICE_FETCHES = 0
UI_NAVIGATION_SNAPSHOTS_CREATED = 0
UI_NAVIGATION_CATCHUP_RUNS = 0
UI_NAVIGATION_POLICY_ACTIVATIONS = 0

==================================================
6. CORRECCIONES PERMITIDAS
==================================================

Si aparece un fallo responsive, corregir exclusivamente:

- CSS .lens-paper-track*;
- layout de LensPaperTrackView;
- tooltip Papel;
- selector Papel;
- breakpoints Papel;
- accesibilidad del renderer Papel.

No ampliar el alcance.

Si no son necesarias correcciones:

- mantener HEAD sin cambios;
- no crear un commit vacío.

Si son necesarias:

crear un único commit local:

fix(lens): finalize responsive paper track charts

No crear tag.
No hacer push.

==================================================
7. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = 0466595
CANONICAL_HEAD_AFTER = <hash/UNCHANGED>

PAPER_PORTFOLIOS_VALIDATED = <número>/5

CHROME_DESKTOP_1440_GREEN = PASS/FAIL
CHROME_DESKTOP_1280_GREEN = PASS/FAIL
CHROME_MOBILE_390_GREEN = PASS/FAIL
CHROME_MOBILE_360_GREEN = PASS/FAIL

MOBILE_TOUCH_TOOLTIP_GREEN = PASS/FAIL
PORTFOLIO_SELECTOR_GREEN = PASS/FAIL
TEMPORAL_SELECTORS_GREEN = PASS/FAIL
KPIS_GREEN = PASS/FAIL
SPY_GREEN = PASS/FAIL
DRAWDOWN_GREEN = PASS/FAIL
CONFIDENCE_GREEN = PASS/FAIL
COVERAGE_GREEN = PASS/FAIL
DISCLAIMER_GREEN = PASS/FAIL

HORIZONTAL_OVERFLOW_CASES = <número>
CLIPPING_CASES = <número>
JAVASCRIPT_ERRORS = <número>

PAPERCHARTHTML_REMOVED = PASS/FAIL
RENDERPAPELTRACK_STATUS = <estado>

UI_NAVIGATION_DATABASE_WRITES = 0
UI_NAVIGATION_PRICE_FETCHES = 0
UI_NAVIGATION_SNAPSHOTS_CREATED = 0
UI_NAVIGATION_POLICY_ACTIVATIONS = 0

NO_ENGINE_CHANGE = PASS/FAIL
NO_SCHEMA_CHANGE = PASS/FAIL
NO_POLICY_ACTIVATED = PASS/FAIL
NO_REAL_ECONOMIC_WRITES = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

LOCAL_COMMIT = <hash/NONE>

GATE_2E_B_FINAL = PASS/FAIL

Detenerse.