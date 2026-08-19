MIZAN · AMPLIACIÓN INCREMENTAL DEL TRACK RECORD
BENCHMARK EQUIPONDERADO · DIFERENCIAL · SIGNIFICANCIA
ORDEN DE TRABAJO PARA CLAUDE CODE

PRIORIDAD

Desarrollo incremental y no destructivo.

Pantalla:

Cartera real
→ Track record

Aplicación:

localhost:3000

==================================================
0. BASELINE Y ALCANCE
==================================================

Baseline local esperado:

CANONICAL_HEAD =
23978af

Commit productivo esperado:

PRODUCTION_HEAD =
1d67c54

Schema esperado:

v31

Estado que debe preservarse:

- Growth C1 activa;
- mínimo ejecutable de 2 USD activo;
- resizing abstracto eliminado;
- benchmark actual correctamente calculado como acumulado desde inicio;
- Gate 2F inactivo;
- cero órdenes automáticas a Wio.

Si HEAD, commit productivo o schema difieren:

- informar del estado real;
- no resetear;
- no descartar commits;
- no sobrescribir trabajo;
- identificar la relación entre los commits;
- solicitar confirmación si no existe una base inequívoca.

Objetivo:

Ampliar el Track record existente sin reescribir ni romper sus cálculos,
componentes o comportamiento actual.

El gráfico existente:

“Valor de la cartera vs S&P 500”

está correcto.

Actualmente compara:

- cartera acumulada desde inception;
- SPY acumulado desde la misma fecha;
- misma base temporal;
- mismo método acumulado.

No corregir ni sustituir esa fórmula salvo que la Tarea 1 demuestre mediante
evidencia reproducible un error diferente.

==================================================
1. EJECUCIÓN POR CHECKPOINTS
==================================================

Esta orden tiene dos checkpoints obligatorios.

CHECKPOINT A — INVENTARIO

En la primera ejecución:

1. crear o cambiar a la rama dedicada;
2. comprobar baseline;
3. tomar captura de referencia;
4. hacer inventario;
5. investigar la Tarea 1 sin modificar código;
6. entregar el informe;
7. detenerse;
8. esperar confirmación expresa de Omar.

No modificar ningún archivo de código en el Checkpoint A.

Se permite únicamente:

- crear la rama;
- leer archivos;
- ejecutar consultas read-only;
- ejecutar tests sin modificar fixtures;
- usar Chrome para capturar la pantalla;
- producir el informe en la respuesta.

No crear todavía commits funcionales.

CHECKPOINT B — IMPLEMENTACIÓN

Solo después de la confirmación expresa de Omar:

- implementar las tareas autorizadas;
- mantener cada tarea en su propio commit;
- ejecutar pruebas;
- validar feature flags;
- comparar capturas;
- desplegar únicamente si Omar lo autoriza.

==================================================
2. REGLAS DE NO-DESTRUCCIÓN (OBLIGATORIAS)
==================================================

Estas reglas tienen prioridad sobre cualquier otra instrucción de este documento. Si alguna entra en conflicto con una tarea, **para y pregunta** en lugar de resolverlo por tu cuenta.

1. **Rama dedicada.** Trabaja en `feature/track-record-benchmarks`. Ningún commit directo a `main`.

2. **Inventario antes de tocar código.** Primera entrega, antes de modificar un solo archivo: lista de los archivos que componen la pantalla Track record, la librería de gráficos que usa el proyecto (detéctala, no la asumas), el módulo donde se calculan los retornos, y el sistema de i18n y de tokens de estilo. **Espera confirmación antes de continuar.**

3. **Solo añadir, nunca reescribir.** Los componentes nuevos van en archivos nuevos. Si necesitas modificar un componente existente, la única modificación permitida es *insertar* — por ejemplo, añadir props opcionales con valor por defecto que preserve exactamente el comportamiento actual.

4. **Prohibido refactorizar.** No renombres variables, no reorganices carpetas, no "limpies" código que funciona, no cambies dependencias, no actualices versiones. Aunque veas algo mejorable, déjalo. Si detectas un problema serio, anótalo en un informe aparte sin tocarlo.

5. **No modifiques las funciones de cálculo existentes.** Las métricas nuevas van en un módulo nuevo. Si una función existente necesita un cambio, propónlo por escrito y espera aprobación.

6. **Feature flags.** Cada elemento nuevo (línea de benchmark, gráfico de diferencial, bloque de significancia) detrás de su propio flag, desactivable sin redeploy.

7. **Bilingüe.** Cada cadena de texto nueva debe existir en ES y EN, usando el sistema de i18n que ya haya en el proyecto.

8. **Respeta el lenguaje visual.** Lee los tokens de color, tipografía y espaciado existentes y úsalos. Tema oscuro, etiquetas en monoespaciada. No introduzcas una paleta nueva ni una librería de UI nueva.

9. **Captura de referencia.** Screenshot de la pantalla antes de empezar. Al terminar, con todos los flags desactivados, la pantalla debe ser **pixel-idéntica** a esa captura. Ese es el criterio de que no has roto nada.

10. **Sin dependencias nuevas** salvo aprobación explícita. Reutiliza la librería de gráficos que ya está en el proyecto.

==================================================
3. CHECKPOINT A · INVENTARIO OBLIGATORIO
==================================================

Crear o cambiar a:

feature/track-record-benchmarks

La rama debe partir del baseline local autorizado.

No hacer cambios funcionales todavía.

Entregar un inventario exacto con rutas, símbolos y responsabilidades.

A. PANTALLA TRACK RECORD

Identificar:

- archivo de entrada;
- componentes;
- funciones de renderizado;
- endpoints consumidos;
- controladores backend;
- consultas;
- tablas;
- cachés;
- scheduler o proceso que produce snapshots;
- código de tooltips;
- tarjetas de ventanas;
- drawdown;
- cabecera de retornos;
- selector de idioma.

B. LIBRERÍA DE GRÁFICOS

Detectar, no asumir:

- nombre;
- versión actual;
- forma en que se importa;
- componentes utilizados;
- soporte existente para:
  - tres series;
  - líneas punteadas;
  - áreas;
  - barras;
  - anotaciones verticales;
  - ejes X compartidos;
  - tooltips personalizados.

No añadir dependencias.

C. CÁLCULOS

Trazar:

- retorno acumulado principal;
- retorno diario;
- TWR;
- variación de valor;
- benchmark SPY;
- spread frente a SPY;
- ventanas 1D, MTD, YTD y 1Y;
- drawdown;
- límites;
- series certificadas;
- neutralización de aportaciones y retiradas.

D. INTERNACIONALIZACIÓN

Identificar:

- archivos;
- estructura de claves;
- mecanismo ES/EN;
- fallback;
- interpolación;
- actualización reactiva al cambiar idioma.

E. ESTILOS

Identificar:

- tokens de color;
- tokens tipográficos;
- espaciado;
- bordes;
- radios;
- tema oscuro;
- tipografía monoespaciada;
- estilos de gráficos;
- colores positivos y negativos.

F. FEATURE FLAGS

Identificar el mecanismo existente para flags:

- fuente;
- persistencia;
- endpoint o configuración;
- comportamiento runtime;
- actualización sin redeploy.

Si no existe un mecanismo compatible, no inventarlo todavía.

Proponer por escrito la extensión mínima necesaria y esperar aprobación.

==================================================
4. CAPTURA DE REFERENCIA
==================================================

Antes de modificar código:

1. abrir Producción en Chrome;
2. seleccionar Cartera real → Track record;
3. seleccionar idioma ES;
4. fijar un viewport reproducible;
5. esperar a que termine la carga;
6. capturar la pantalla completa;
7. registrar:
   - viewport;
   - device scale factor;
   - idioma;
   - fecha/hora;
   - HEAD productivo;
   - schema;
   - cartera seleccionada;
   - flags actuales.

Tomar también captura en EN si el cambio de idioma modifica la disposición.

Guardar las capturas en una ubicación no productiva coherente con las
convenciones existentes del repositorio.

Si guardar la captura implica crear un archivo en el repositorio, no hacerlo
todavía: conservarla como artefacto de la sesión y proponer su ruta.

==================================================
5. TAREA 1 · DIAGNÓSTICO SIN MODIFICACIONES
==================================================

La cabecera del gráfico muestra aproximadamente:

cartera =
+1,13 %

Las tarjetas muestran:

En el mes =
+0,49 %

La fecha inicial visible parece ser:

2026-07-01

SPY parece coincidir entre ambos lugares.

Existe una diferencia de:

64 puntos básicos

Determinar el origen exacto.

Hipótesis que debe comprobarse, no asumirse:

- una cifra es TWR;
- otra es cambio de valor o retorno ponderado por dinero;
- la divergencia procede de movimientos manuales del 24 de julio de 2026.

Trazar para ambas cifras:

- componente UI;
- campo API;
- endpoint;
- función;
- tabla;
- filas fuente;
- fecha base;
- fecha final;
- tratamiento de flujos;
- fórmula;
- redondeo.

Entregar para cada cifra:

METRIC_LOCATION =
<header/window card>

METRIC_NAME =
<nombre>

METHOD =
<TWR/MWR/value change/otro>

BASE_DATE =
<fecha>

END_DATE =
<fecha>

SOURCE =
<tabla/campo>

FORMULA =
<fórmula>

EXTERNAL_FLOWS_TREATMENT =
<descripción>

RESULT_BEFORE_ROUNDING =
<valor>

DISPLAYED_RESULT =
<valor>

Comprobar específicamente los movimientos del:

2026-07-24

y demostrar si explican la diferencia.

Comprobar también la tarjeta:

1 día

Determinar si es:

- retorno exclusivamente diario;
- acumulado de una ventana;
- diferencia entre dos snapshots;
- otro método.

No modificar etiquetas ni fórmulas en Checkpoint A.

==================================================
6. CLASIFICACIÓN DE LA DISCREPANCIA
==================================================

Clasificar la Tarea 1 como:

A. BOTH_CORRECT_DIFFERENT_METHODS

Ambas cifras son correctas, pero miden conceptos distintos.

Propuesta posterior:

- conservar ambas;
- etiquetar el método;
- indicar ventana;
- añadir explicación ES/EN.

B. ONE_METRIC_INCORRECT

Una cifra usa una base, flujo o fórmula incorrectos.

Entregar:

- causa raíz;
- alcance;
- propuesta mínima;
- función existente que necesitaría modificación;
- razón por la que no puede corregirse únicamente de forma aditiva.

No corregir sin aprobación debido a la regla 5.

C. SAME_METHOD_DIFFERENT_WINDOW

Las cifras usan el mismo método, pero no la misma fecha inicial o final.

Proponer etiquetas temporales exactas.

D. DISPLAY_OR_ROUNDING_PROBLEM

Los datos fuente coinciden, pero la UI etiqueta o redondea incorrectamente.

Proponer la corrección aditiva mínima.

==================================================
7. ENTREGA DEL CHECKPOINT A
==================================================

Entregar exactamente:

BRANCH =
feature/track-record-benchmarks

SOURCE_HEAD =
<hash>

PRODUCTION_HEAD =
<hash>

PRODUCTION_SCHEMA =
<versión>

WORKTREE_CLEAN_BEFORE =
YES/NO

FILES_MODIFIED =
0

REFERENCE_SCREENSHOT =
<artefacto/ruta propuesta>

TRACK_RECORD_FILES =
<lista>

CHART_LIBRARY =
<nombre y versión>

CHART_CAPABILITIES =
<lista relevante>

RETURN_CALCULATION_MODULES =
<lista>

TRACK_RECORD_ENDPOINTS =
<lista>

TRACK_RECORD_TABLES =
<lista>

I18N_SYSTEM =
<archivos y mecanismo>

STYLE_TOKEN_SYSTEM =
<archivos y mecanismo>

FEATURE_FLAG_SYSTEM =
<archivos y mecanismo/NOT_FOUND>

TASK_1_CLASSIFICATION =
A/B/C/D

HEADER_RETURN_METHOD =
<método>

WINDOW_RETURN_METHOD =
<método>

HEADER_BASE_DATE =
<fecha>

WINDOW_BASE_DATE =
<fecha>

JULY_24_MANUAL_INTERVENTION_EFFECT =
<importe y explicación>

ONE_DAY_CARD_METHOD =
<método>

EXISTING_FUNCTION_CHANGE_REQUIRED =
YES/NO

PROPOSED_INSERTION_POINTS =
<archivos, símbolos y propósito>

NEW_FILES_PROPOSED =
<lista>

RISKS_FOUND_BUT_NOT_TOUCHED =
<lista/NONE>

EXACT_QUESTIONS_REQUIRING_APPROVAL =
<lista/NONE>

CHECKPOINT_A_STATUS =
COMPLETE

Detenerse después de esta entrega.

No implementar las tareas 2 a 6.

Esperar confirmación expresa de Omar.

==================================================
8. CHECKPOINT B · TAREA 2
BENCHMARK DEL UNIVERSO EQUIPONDERADO
==================================================

Solo ejecutar después de aprobación.

Construir una serie sintética usando las acciones del universo analizado.

No hardcodear 123.

Resolver dinámicamente la composición certificada correspondiente a cada
sesión.

Composición inicial esperada:

123 acciones

Metodología:

- universo analizado;
- equiponderación diaria;
- rebalanceo diario;
- cierre oficial;
- encadenamiento geométrico.

Para cada sesión t:

1. identificar miembros con precio válido en t−1 y t;
2. excluir valores sin cotización válida;
3. redistribuir pesos por igual entre los valores elegibles;
4. calcular el retorno diario equiponderado;
5. encadenar geométricamente.

daily_equal_weight_return(t) =
SUM(valid_security_return(t)) / valid_security_count(t)

index_level(t) =
index_level(t-1) × (1 + daily_equal_weight_return(t))

cumulative_return(t) =
index_level(t) / base_index_level - 1

No imputar retorno cero a un valor sin precio.

Persistir o exponer:

- total universe members;
- eligible members;
- excluded members;
- exclusiones y razones;
- fecha;
- price session;
- methodology version;
- content hash.

Fecha base solicitada:

2026-06-30

Antes de utilizarla, verificar que existen:

- composición certificada;
- precios certificados de cierre;
- datos suficientes de las acciones.

Si no existe historia certificada el 30 de junio:

- no inventarla;
- no retrotraer la composición actual silenciosamente;
- informar;
- proponer INSUFFICIENT_HISTORY o un backfill explícito;
- esperar aprobación si se necesita un backfill.

No confundir esta limitación con la del track real, que comenzó el 1 de julio.

La serie del universo puede tener base 30 de junio solo si sus propios datos
certificados lo permiten.

Renderizado con flag independiente:

TRACK_RECORD_UNIVERSE_EW_BENCHMARK

Cuando está false:

- gráfico actual pixel-idéntico;
- dos series actuales intactas;
- misma leyenda;
- mismo tooltip;
- mismo orden.

Cuando está true:

- cartera como serie principal;
- universo EW como benchmark principal sólido;
- SPY como referencia secundaria gris punteada tenue;
- tooltip con tres series;
- leyenda bilingüe;
- metodología y base visibles.

==================================================
9. CHECKPOINT B · TAREA 3
CURVA DE DIFERENCIAL
==================================================

Crear un componente independiente en un archivo nuevo.

Flag:

TRACK_RECORD_DIFFERENTIAL_CHART

Con false:

- no renderizar el componente;
- cero cambios de layout;
- pixel-idéntico al baseline.

Con true:

Panel superior:

cumulative_relative_performance(t) =
portfolio_cumulative_return(t)
-
selected_benchmark_cumulative_return(t)

- unidad: puntos porcentuales;
- área suave;
- línea de cero;
- mismo eje temporal.

Panel inferior:

daily_relative_performance(t) =
portfolio_daily_return(t)
-
selected_benchmark_daily_return(t)

- barras verdes si > 0;
- rojas si < 0;
- neutras si = 0;
- eje X compartido.

Selector:

- universo EW;
- SPY.

Métricas:

batting_average =
winning_sessions / comparable_sessions

alpha_excluding_best_session =
geometric_relative_result_excluding_max_daily_relative_session

No restar simplemente el mayor punto acumulado.

Calcular nuevamente el encadenamiento diario sin la mejor sesión relativa.

Mostrar:

- batting average;
- alfa sin mejor sesión;
- mejor sesión relativa;
- peor sesión relativa;
- fechas;
- N comparable.

==================================================
10. CHECKPOINT B · TAREA 4
EVENTOS DE INTERVENCIÓN MANUAL
==================================================

Crear un archivo o tabla de configuración nueva, según los patrones existentes.

Modelo mínimo:

- event_id;
- date;
- portfolio_id;
- type;
- description_i18n_key;
- optional metadata;
- active.

Evento inicial:

date =
2026-07-24

portfolio =
crecimiento o cartera afectada verificada

type =
MANUAL_INTERVENTION

No asumir el alcance exacto sin consultar los movimientos y la Tarea 1.

Renderizar:

- marca vertical;
- anotación;
- tooltip;
- descripción ES/EN.

Flag propio:

TRACK_RECORD_MANUAL_EVENTS

Con false:

- no alterar el gráfico actual.

==================================================
11. CHECKPOINT B · TAREA 5
SIGNIFICANCIA ESTADÍSTICA
==================================================

Crear módulo de cálculo nuevo.

No modificar funciones de retorno existentes.

Flag:

TRACK_RECORD_SIGNIFICANCE

Umbral configurable inicial:

MIN_SIGNIFICANCE_SESSIONS =
60

Métricas:

- diferencial acumulado;
- N;
- tracking error;
- information ratio;
- error estándar del alfa anualizado;
- estado cualitativo.

No mostrar cifras anualizadas si:

N < MIN_SIGNIFICANCE_SESSIONS

Mostrar:

—
Historia insuficiente

en ES, y su equivalente en EN.

No calcular ni mostrar una cifra anualizada engañosa.

Definir y documentar:

tracking_error_annualized =
stddev(daily_portfolio_return - daily_benchmark_return)
× sqrt(trading_sessions_per_year)

information_ratio =
annualized_active_return / tracking_error_annualized

El método de annualized_active_return y del error estándar debe quedar
explícito y probado.

No utilizar aproximaciones incompatibles con el historial disponible.

==================================================
12. CHECKPOINT B · TAREA 6
AJUSTES ADITIVOS
==================================================

A. TARJETAS DE VENTANA

Conservar:

- componentes;
- estructura;
- estilos;
- comportamiento existente cuando el flag esté desactivado.

Añadir estado de historia insuficiente.

Mientras no exista historia propia:

Desde inicio:
- activa.

En el mes:
- activa solo si existe ancla válida anterior al periodo.

En el año:
- activa solo si existe ancla válida anterior al año.

1 año:
- activa solo si existe una ventana completa de un año.

Si no:

- mostrar “—”;
- mostrar historia insuficiente;
- no repetir el retorno desde inicio como sustituto.

Flag:

TRACK_RECORD_WINDOW_AVAILABILITY

B. DRAWDOWN

Añadir sin modificar:

- semáforo;
- límite −40,20 %;
- drawdown actual;
- margen actual.

Nuevos datos:

- drawdown del benchmark seleccionado;
- volatilidad realizada anualizada.

Flag:

TRACK_RECORD_DRAWDOWN_CONTEXT

Definir:

- benchmark utilizado;
- ventana;
- sesiones;
- anualización;
- insufficient history.

==================================================
13. FEATURE FLAGS
==================================================

Cada elemento debe disponer de un flag independiente:

TRACK_RECORD_UNIVERSE_EW_BENCHMARK
TRACK_RECORD_DIFFERENTIAL_CHART
TRACK_RECORD_MANUAL_EVENTS
TRACK_RECORD_SIGNIFICANCE
TRACK_RECORD_WINDOW_AVAILABILITY
TRACK_RECORD_DRAWDOWN_CONTEXT

Los flags deben:

- usar el sistema existente;
- poder desactivarse sin redeploy;
- tener default false inicialmente;
- no cambiar el payload o layout existente cuando estén desactivados;
- permitir activación individual;
- ser visibles en diagnóstico/env-info si el sistema lo soporta.

No activar todos simultáneamente en Producción sin preview y aprobación.

==================================================
14. COMMITS
==================================================

Después del Checkpoint A y de la aprobación:

Commit 1:
feat(track): add equal-weight universe benchmark

Commit 2:
feat(track): add relative performance analysis

Commit 3:
feat(track): annotate manual portfolio interventions

Commit 4:
feat(track): add statistical significance context

Commit 5:
feat(track): add window availability and benchmark risk context

Si la Tarea 1 requiere una corrección autorizada:

Commit independiente previo:

fix(track): clarify return methods and periods

o el título exacto correspondiente a la causa real.

No mezclar tareas en un commit.

No hacer commits directos a main.

No hacer push salvo autorización.

No crear tag.

==================================================
15. PRUEBAS
==================================================

Crear pruebas para el módulo nuevo.

Benchmark EW con retornos conocidos:

Ejemplo:

Día 1:
+1 %

Día 2:
+1 %

Acumulado esperado:

(1.01 × 1.01) - 1 =
2,01 %

No aceptar:

2,00 %

Datos ausentes:

- excluir valor sin cotización;
- redistribuir pesos;
- no imputar cero;
- registrar exclusión.

Probar además:

- universo vacío;
- un único miembro;
- miembro incorporado;
- miembro retirado;
- precio anterior ausente;
- precio actual ausente;
- retorno extremo;
- input duplicado;
- sesiones desordenadas;
- idempotencia;
- hash estable;
- base exacta;
- misma fecha entre series comparadas.

Diferencial:

- acumulado correcto;
- diario correcto;
- batting average;
- mejor y peor sesión;
- exclusión de la mejor sesión;
- benchmark selectable.

Significancia:

- N inferior a 60;
- N igual a 60;
- tracking error cero;
- historia incompleta;
- anualización oculta con N insuficiente.

Flags:

- todos false;
- uno activado;
- combinaciones;
- runtime toggle;
- sin cambio de payload legado cuando corresponda.

==================================================
16. VALIDACIÓN PIXEL-IDÉNTICA
==================================================

Con todos los flags false:

- utilizar el mismo viewport;
- mismo idioma;
- mismo dataset o snapshot;
- misma versión de navegador;
- misma escala;
- misma cartera.

Comparar contra la captura inicial.

Resultado obligatorio:

PIXEL_DIFF =
0

Si datos dinámicos impiden una comparación directa:

- congelar mediante fixture o copia;
- no ocultar diferencias;
- distinguir cambio de datos de cambio de layout.

No considerar aprobado un “se parece”.

Debe existir evidencia reproducible de identidad visual.

==================================================
17. INTERNACIONALIZACIÓN
==================================================

Toda cadena nueva debe tener ES y EN:

- universo equiponderado;
- referencia secundaria;
- diferencial acumulado;
- diferencial diario;
- batting average;
- alfa sin mejor sesión;
- mejor sesión;
- peor sesión;
- intervención manual;
- tracking error;
- information ratio;
- error estándar;
- historia insuficiente;
- desde inicio;
- método TWR;
- ponderado por dinero;
- acumulado;
- diario;
- benchmark drawdown;
- volatilidad realizada.

No insertar textos directos fuera del sistema i18n.

==================================================
18. SEGURIDAD Y NO REGRESIÓN
==================================================

No modificar:

- Growth C1;
- aportaciones;
- recomendaciones;
- Book;
- holdings;
- cash;
- NAV;
- fills Wio;
- materialidad de 2 USD;
- metodología C0/C1;
- scheduler de aportaciones;
- Gate 2F;
- endpoints de resizing eliminados.

Ejecutar regresiones de:

- Track record;
- benchmark;
- Growth C1;
- financial core;
- Book;
- holdings;
- cash;
- NAV;
- dashboard;
- i18n;
- feature flags;
- repository integrity.

No realizar escrituras económicas durante las pruebas.

No enviar órdenes a Wio.

==================================================
19. INFORME DE PROBLEMAS NO TOCADOS
==================================================

Crear un informe separado con:

- archivo;
- símbolo;
- problema;
- severidad;
- impacto;
- reproducción;
- recomendación.

No corregir esos problemas dentro de este alcance.

No mezclar deuda técnica con las tareas autorizadas.

==================================================
20. ENTREGA FINAL
==================================================

BRANCH =
feature/track-record-benchmarks

BASE_HEAD =
<hash>

FINAL_HEAD =
<hash>

PRODUCTION_SCHEMA =
v31

TASK_1_ROOT_CAUSE =
<descripción>

TASK_1_CLASSIFICATION =
A/B/C/D

HEADER_RETURN_METHOD =
<método>

WINDOW_RETURN_METHOD =
<método>

ONE_DAY_CARD_METHOD =
<método>

UNIVERSE_EW_BENCHMARK =
PASS/FAIL

UNIVERSE_MEMBERS_EXPECTED =
<número dinámico>

UNIVERSE_BASE_DATE =
<fecha/INSUFFICIENT_HISTORY>

MISSING_PRICE_POLICY =
EXCLUDE_AND_REWEIGHT

GEOMETRIC_CHAINING =
PASS/FAIL

DIFFERENTIAL_CHART =
PASS/FAIL

BATTING_AVERAGE =
PASS/FAIL

ALPHA_WITHOUT_BEST_SESSION =
PASS/FAIL

MANUAL_EVENT_2026_07_24 =
PASS/FAIL

STATISTICAL_SIGNIFICANCE =
PASS/FAIL

MIN_SIGNIFICANCE_SESSIONS =
60

ANNUALIZED_VALUES_HIDDEN_BELOW_THRESHOLD =
PASS/FAIL

WINDOW_AVAILABILITY =
PASS/FAIL

BENCHMARK_DRAWDOWN =
PASS/FAIL

REALIZED_VOLATILITY =
PASS/FAIL

FEATURE_FLAGS_RUNTIME =
PASS/FAIL

ALL_NEW_FLAGS_DEFAULT_FALSE =
PASS/FAIL

SPANISH_TRANSLATIONS =
PASS/FAIL

ENGLISH_TRANSLATIONS =
PASS/FAIL

NEW_DEPENDENCIES =
0

EXISTING_CALCULATION_FUNCTIONS_MODIFIED =
0

FORBIDDEN_REFACTORS =
0

PIXEL_DIFF_WITH_FLAGS_OFF =
0

TRACK_RECORD_TESTS =
PASS/FAIL

REGRESSION_TESTS =
PASS/FAIL

PRODUCTION_ECONOMIC_WRITES =
0

PRODUCTION_BROKER_ORDERS =
0

GATE_2F_REMAINS_INACTIVE =
PASS/FAIL

PUSH_PERFORMED =
NO

TAG_CREATED =
NO

PROBLEMS_FOUND_NOT_TOUCHED =
<lista/NONE>

EXACT_REMAINING_BLOCKERS =
NONE/<lista>

Detenerse después de entregar el resultado autorizado.

No desplegar ni activar flags en Producción sin aprobación expresa de Omar.