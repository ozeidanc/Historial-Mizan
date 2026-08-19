CHECKPOINT A APROBADO

Queda aprobado el inventario y el diagnóstico.

BRANCH =
feature/track-record-benchmarks

SOURCE_HEAD =
23978af

PRODUCTION_HEAD =
1d67c54

PRODUCTION_SCHEMA =
v31

Autorizo comenzar el Checkpoint B del Track record con las decisiones que
siguen.

No implementar todavía Cockpit V2 en esta rama.

==================================================
1. RESPUESTA Q1 · FEATURE FLAGS
==================================================

Aprobar un mecanismo runtime centralizado en backend.

Implementar:

GET /internal/ui-flags

y una operación interna protegida para modificar los overrides runtime.

Requisitos:

- los flags de entorno existentes siguen siendo los defaults;
- los overrides runtime se aplican por encima;
- todos los flags nuevos tienen default false;
- la UI consulta el estado efectivo;
- pueden activarse y desactivarse sin redeploy y sin reinicio;
- GET es read-only;
- la escritura solo puede aceptarse desde localhost o mediante el mecanismo
  interno de autenticación ya existente;
- registrar cambios en el log de auditoría;
- no usar querystring ni localStorage como fuente de autoridad;
- no añadir dependencias;
- no crear una interfaz pública de administración.

Los overrides pueden ser in-memory si el patrón actual no dispone de
persistencia compatible.

Después de un reinicio pueden volver a los defaults de entorno, siempre que
esto quede documentado.

No crear una migración de schema exclusivamente para los flags.

==================================================
2. RESPUESTA Q2 · BASE DEL UNIVERSO EW
==================================================

Verificar primero si existen composición y precios certificados del universo
para el cierre del:

2026-06-30

No autorizar todavía ningún backfill.

Si existen datos certificados suficientes:

UNIVERSE_EW_CANONICAL_BASE_DATE =
2026-06-30

Si no existen:

UNIVERSE_EW_CANONICAL_STATUS =
INSUFFICIENT_HISTORY

No:

- inventar composición;
- retrotraer el universo actual;
- utilizar precios no certificados;
- hacer backfill sin autorización posterior.

Distinción obligatoria:

A. BASE CANÓNICA DEL ÍNDICE EW

Puede ser 2026-06-30 si sus propios datos lo permiten.

B. BASE DE COMPARACIÓN EN PANTALLA

Todas las series visibles deben compartir una fecha base comparable.

Como el Track real comienza el 2026-07-01, el gráfico debe usar:

DISPLAY_COMPARISON_BASE_DATE =
2026-07-01

salvo que en el futuro exista un snapshot certificado de la cartera para
2026-06-30.

Si el índice EW comienza el 2026-06-30, conservar su nivel canónico, pero
rebasar su retorno visible a cero el 2026-07-01 para compararlo con la cartera.

En la fecha base visible debe cumplirse exactamente:

portfolio_display_return =
0

universe_ew_display_return =
0

spy_display_return =
0

No comparar una cartera desde el 1 de julio contra un benchmark acumulado
desde el 30 de junio.

==================================================
3. RESPUESTA Q3 · DISCREPANCIA DE 64 PB
==================================================

Confirmo:

TASK_1_CLASSIFICATION =
C_SAME_METHOD_DIFFERENT_WINDOW

Autorizar únicamente la corrección aditiva de etiquetas del comportamiento
existente.

Header:

“P&L total sobre coste · ponderado por dinero · desde la compra”

Ventana MTD:

“Variación del P&L sobre coste · desde 01/07/2026”

Tarjeta 1 día:

“Variación de tu_pct desde el snapshot anterior”

Crear equivalentes exactos en EN.

No presentar la tarjeta 1 día como “retorno diario de mercado”, porque no lo
es.

No modificar:

- valuacion.mjs;
- metricasVentanas;
- tu_pct;
- revisiones históricas;
- cálculos legacy.

==================================================
4. DECISIÓN METODOLÓGICA PARA LAS NUEVAS ANALÍTICAS
==================================================

El diagnóstico demuestra que tu_pct es ponderado por dinero y que la entrada
de capital/posiciones del 24 de julio produjo:

tu_pct:
2,68 %
→
0,39 %

sin representar una caída equivalente de mercado.

Por tanto, no autorizar comparar el tu_pct legacy contra el universo EW para
evaluar la selección del modelo.

Las nuevas funciones de:

- benchmark EW;
- diferencial acumulado;
- diferencial diario;
- batting average;
- alfa sin mejor sesión;
- tracking error;
- information ratio;
- volatilidad;
- significancia;

deben usar la serie canónica neutral a flujos:

portfolio_return_snapshot

o el mecanismo unitizado/TWR canónico equivalente ya existente.

No modificar las funciones existentes.

Crear adaptadores nuevos y read-only.

Declarar claramente:

LEGACY_ACCOUNT_PNL_METHOD =
MONEY_WEIGHTED_PNL_ON_COST

NEW_BENCHMARK_ANALYTICS_METHOD =
TIME_WEIGHTED_RETURN

La UI debe diferenciar:

- “P&L de la cuenta, ponderado por dinero”;
- “Rendimiento TWR, neutral a aportaciones y retiradas”.

Nunca mostrar un diferencial calculado como:

MWR portfolio
−
geometric benchmark return

porque sería metodológicamente inconsistente.

==================================================
5. GRÁFICO LEGACY Y GRÁFICO AMPLIADO
==================================================

Con todos los flags false:

- mantener exactamente el gráfico actual;
- conservar tu_pct;
- conservar SPY;
- conservar cabecera;
- conservar tooltip;
- PIXEL_DIFF = 0.

Con TRACK_RECORD_UNIVERSE_EW_BENCHMARK=true:

- usar la serie TWR de cartera;
- usar universo EW geométrico;
- usar SPY acumulado;
- las tres series con la misma base visible;
- etiquetar explícitamente “TWR”;
- no reutilizar tu_pct como si fuera TWR.

El gráfico ampliado no debe cambiar ni reinterpretar silenciosamente el
gráfico legacy.

Si es necesario, mostrar una nota:

“Vista comparativa neutral a flujos externos (TWR)”.

==================================================
6. INTERVENCIÓN DEL 24 DE JULIO
==================================================

Autorizar el evento:

date =
2026-07-24

portfolio_id =
crecimiento

type =
MANUAL_INTERVENTION

Descripción ES:

“Intervención manual: incorporación de posiciones ajena a la ejecución
automática del modelo.”

Descripción EN:

“Manual intervention: positions added outside the model’s automated
execution.”

Verificar el portfolio_id exacto antes de persistirlo.

La anotación no debe afirmar que hubo una aportación si los movimientos
representan traslado o incorporación de posiciones.

Mostrar el impacto observado de forma informativa:

invested_cost:
1.026,64
→
2.761,27

No atribuir causalidad de rentabilidad al evento.

==================================================
7. TARJETAS DE VENTANA
==================================================

Autorizar el comportamiento propuesto:

- Desde inicio: activa;
- MTD: activa solo con ancla válida anterior al periodo;
- YTD: activa solo con ancla válida anterior al año;
- 1 año: activa solo con ventana completa;
- sin historia: “— · Historia insuficiente”.

No repetir el retorno desde inicio como sustituto.

Las tarjetas legacy permanecen intactas con el flag false.

==================================================
8. ARCHIVOS FRONTEND NUEVOS
==================================================

La SPA actual es monolítica, pero la regla de no-destrucción exige que los
componentes nuevos estén en archivos nuevos.

Por tanto, no añadir toda la implementación nueva dentro de
mizan-dashboard.html.

Crear módulos frontend nuevos siguiendo el mecanismo compatible más sencillo,
sin dependencias:

- track-record-flags.js;
- track-record-ew-chart.js;
- track-record-differential.js;
- track-record-events.js;
- track-record-significance.js;
- track-record-window-context.js;
- track-record-drawdown-context.js.

Los nombres pueden adaptarse a las convenciones reales.

En mizan-dashboard.html solo se permiten inserciones mínimas:

- carga de módulos;
- contenedores opcionales;
- llamada de inicialización;
- props/defaults compatibles.

Con flags false, los módulos no deben modificar el DOM legacy.

==================================================
9. CAPTURAS DE REFERENCIA
==================================================

Autorizar guardar los artefactos de baseline en:

mizan-ops/ui-baselines/track-record-2026-07-28/

Incluir:

- Track record ES;
- Track record EN;
- Cockpit ES;
- Cockpit EN;
- manifest con viewport, idioma, HEAD, schema, cartera y timestamp.

Las capturas de Temp son efímeras.

Copiarlas antes de que se pierdan.

No mezclar las capturas con un commit funcional.

Crear un commit documental separado si las convenciones del repositorio lo
permiten.

==================================================
10. COCKPIT V2
==================================================

La auditoría del Cockpit queda aprobada.

No implementar Cockpit V2 en:

feature/track-record-benchmarks

Después de cerrar el Track record, crear:

feature/cockpit-v2

desde el HEAD integrado que corresponda.

Antes de implementar Cockpit V2, entregar un checkpoint de diseño con:

- wireframe desktop completo;
- wireframe viewport pequeño;
- componentes propuestos;
- campos exactos por componente;
- información eliminada por duplicidad;
- información movida a detalles expandibles;
- enlaces de acciones requeridas;
- contrato de COCKPIT_V2;
- capturas mockup o preview no productivo.

COCKPIT_V2 debe reutilizar el nuevo resumen TWR del Track record, pero no
duplicar los gráficos analíticos completos.

==================================================
11. ORDEN DE IMPLEMENTACIÓN DEL TRACK RECORD
==================================================

Implementar en este orden:

COMMIT 1

fix(track): clarify return methods and periods

Solo etiquetas y estados de historia insuficiente autorizados.

COMMIT 2

feat(track): add runtime UI feature flags

COMMIT 3

feat(track): add equal-weight universe benchmark

COMMIT 4

feat(track): add relative performance analysis

COMMIT 5

feat(track): annotate manual portfolio interventions

COMMIT 6

feat(track): add statistical significance context

COMMIT 7

feat(track): add benchmark drawdown and volatility context

No mezclar Cockpit V2.

No modificar funciones de cálculo legacy.

No hacer push.

No crear tag.

==================================================
12. CHECKPOINT INTERMEDIO OBLIGATORIO
==================================================

Antes de desplegar o activar flags, entregar:

UNIVERSE_2026_06_30_DATA_AVAILABLE =
YES/NO

UNIVERSE_EW_CANONICAL_BASE_DATE =
<fecha/INSUFFICIENT_HISTORY>

DISPLAY_COMPARISON_BASE_DATE =
2026-07-01

PORTFOLIO_ANALYTICS_METHOD =
TWR

LEGACY_HEADER_METHOD =
MWR_PNL_ON_COST

ALL_DISPLAY_SERIES_SHARE_BASE =
PASS/FAIL

RUNTIME_FLAGS =
PASS/FAIL

ALL_FLAGS_DEFAULT_FALSE =
PASS/FAIL

PIXEL_DIFF_ALL_FLAGS_FALSE =
<número>

TRACK_RECORD_ES =
PASS/FAIL

TRACK_RECORD_EN =
PASS/FAIL

NO_EXISTING_CALCULATION_FUNCTIONS_MODIFIED =
PASS/FAIL

NEW_DEPENDENCIES =
0

COCKPIT_V2_IMPLEMENTED_IN_THIS_BRANCH =
NO

Detenerse en ese checkpoint.

No desplegar ni activar flags en Producción hasta recibir aprobación expresa.