==================================================
ANEXO · AUDITORÍA Y REDISEÑO DEL COCKPIT
==================================================

Además del Track record, incluir en el Checkpoint A una auditoría read-only
del Cockpit actual.

En este checkpoint:

- no modificar el Cockpit;
- no ocultar módulos;
- no mover información;
- no cambiar endpoints;
- no cambiar cálculos;
- no implementar todavía el rediseño.

El objetivo es identificar por qué la información está dispersa y proponer
una arquitectura clara antes de tocar código.

==================================================
A. SEPARACIÓN DEL TRABAJO
==================================================

La inspección del Cockpit puede realizarse durante el Checkpoint A de:

feature/track-record-benchmarks

Pero su implementación no debe mezclarse con esa rama.

Después de aprobar el diseño, crear una rama independiente:

feature/cockpit-v2

La implementación del Cockpit debe quedar detrás de:

COCKPIT_V2

Con el flag false:

- debe renderizarse el Cockpit actual;
- no debe cambiar ningún cálculo;
- no debe cambiar ningún endpoint;
- no debe cambiar el layout actual;
- el comportamiento debe permanecer intacto.

No desplegar ni activar COCKPIT_V2 sin aprobación expresa de Omar.

==================================================
B. CAPTURA DE REFERENCIA
==================================================

Antes de modificar código:

1. abrir el Cockpit actual en Producción;
2. seleccionar la cartera real;
3. capturar la vista completa en ES;
4. capturarla en EN;
5. utilizar el mismo viewport reproducible del Track record;
6. registrar scroll, viewport, escala, HEAD, schema y cartera;
7. capturar también estados con alertas, acciones pendientes o datos
   incompletos si pueden reproducirse sin escrituras.

Entregar:

COCKPIT_REFERENCE_SCREENSHOT_ES =
<artefacto>

COCKPIT_REFERENCE_SCREENSHOT_EN =
<artefacto>

No guardar capturas en el repositorio antes de confirmar la convención
existente.

==================================================
C. INVENTARIO DEL COCKPIT
==================================================

Identificar con rutas y símbolos:

- archivo de entrada;
- funciones de renderizado;
- componentes o secciones;
- tarjetas;
- tablas;
- gráficas;
- modales;
- acciones;
- navegación;
- endpoints;
- campos consumidos;
- tablas backend;
- cálculos;
- cachés;
- scheduler;
- sistema de actualización;
- i18n;
- estilos;
- estados de carga;
- estados de error;
- estados vacíos.

Para cada bloque visible entregar:

SECTION_ID =
<id>

VISIBLE_TITLE =
<título>

PURPOSE =
<qué pregunta responde>

DATA_SOURCE =
<endpoint/campo>

CALCULATION_SOURCE =
<función/NONE>

UPDATE_FREQUENCY =
<frecuencia>

USER_ACTION =
<acción/NONE>

DUPLICATES_INFORMATION_FROM =
<sección/NONE>

CONFLICTS_WITH =
<sección/NONE>

OPERATIONAL_IMPORTANCE =
CRITICAL/HIGH/MEDIUM/LOW

==================================================
D. DIAGNÓSTICO DE INFORMACIÓN DISPERSA
==================================================

Auditar específicamente:

1. información duplicada en varias zonas;
2. métricas similares con fórmulas diferentes;
3. datos sin ventana temporal visible;
4. datos sin método de cálculo visible;
5. acciones importantes lejos del dato que las origina;
6. alertas mezcladas con información secundaria;
7. estados operativos mezclados con métricas analíticas;
8. datos técnicos ocupando espacio principal;
9. componentes cuyo propósito no es evidente;
10. información necesaria que solo aparece en otras pantallas;
11. valores sin fecha de actualización;
12. mensajes que parecen acciones pero no lo son;
13. botones con consecuencias económicas insuficientemente claras;
14. diferencias entre ES y EN;
15. problemas en viewport pequeño.

No corregirlos todavía.

Clasificar cada problema como:

- DUPLICATE;
- SCATTERED;
- AMBIGUOUS;
- INCONSISTENT;
- LOW_PRIORITY_NOISE;
- MISSING_CONTEXT;
- ACTION_DISCOVERY;
- VISUAL_HIERARCHY;
- RESPONSIVE;
- ACCESSIBILITY;
- DATA_QUALITY.

==================================================
E. PREGUNTAS QUE DEBE RESPONDER EL COCKPIT
==================================================

Evaluar si el Cockpit permite responder rápidamente y en este orden:

1. ¿Está Mizan operativo y actualizado?
2. ¿Hay alguna acción urgente que Omar deba realizar?
3. ¿Existen recomendaciones pendientes?
4. ¿Existen fills Wio pendientes de registrar?
5. ¿Existen problemas de conciliación?
6. ¿Cuál es el estado actual de la cartera?
7. ¿Cuánto cash está disponible?
8. ¿Cuál es el NAV?
9. ¿Cuál es el rendimiento del periodo relevante?
10. ¿Cuál es el riesgo actual?
11. ¿Cuándo será la próxima revisión?
12. ¿Existen procesos esperando mercado, precios o scheduler?
13. ¿Hay algún bloqueo que impida operar?
14. ¿Qué datos son meramente informativos?

Para cada pregunta entregar:

ANSWERABLE =
YES/NO/PARTIAL

CURRENT_LOCATION =
<ubicación>

CLICKS_REQUIRED =
<número>

PROBLEM =
<descripción/NONE>

==================================================
F. ARQUITECTURA PROPUESTA
==================================================

Proponer una arquitectura de información, sin implementarla todavía.

La propuesta debe priorizar aproximadamente:

1. ESTADO OPERATIVO

- salud de Mizan;
- última actualización;
- sesión de mercado;
- datos disponibles;
- bloqueos;
- conciliación.

2. ACCIONES REQUERIDAS

- recomendaciones pendientes;
- fills Wio pendientes;
- aportaciones pendientes;
- errores recuperables;
- procesos que necesitan intervención.

3. RESUMEN DE CARTERA

- NAV;
- cash;
- valor invertido;
- holdings;
- exposición;
- fecha de valoración.

4. RENDIMIENTO

- retorno;
- benchmark;
- diferencial;
- drawdown;
- enlace al Track record ampliado.

5. PRÓXIMOS EVENTOS

- siguiente revisión;
- scheduler;
- recálculos;
- apertura/cierre de mercado;
- solicitudes pendientes.

6. INFORMACIÓN SECUNDARIA

- detalles técnicos;
- hashes;
- diagnósticos;
- procesos internos;
- información expandible.

Esto es una orientación, no una estructura impuesta.

Code debe adaptar la propuesta al inventario real y explicar cualquier cambio.

==================================================
G. JERARQUÍA VISUAL
==================================================

Proponer tres niveles:

NIVEL 1 — URGENTE

Solo información que requiera acción inmediata o represente un bloqueo:

- errores;
- datos no reconciliados;
- fills pendientes;
- revisión accionable;
- proceso fallido;
- riesgo crítico.

NIVEL 2 — ESTADO PRINCIPAL

- NAV;
- cash;
- cartera invertida;
- rendimiento;
- benchmark;
- drawdown;
- próxima revisión.

NIVEL 3 — DETALLE

- metodología;
- fuentes;
- timestamps completos;
- hashes;
- diagnósticos;
- auditoría;
- procesos internos.

No utilizar color de alerta para información normal.

No convertir todas las tarjetas en elementos de igual importancia.

==================================================
H. ACCIONES ECONÓMICAS
==================================================

Inventariar todas las acciones del Cockpit que puedan afectar:

- Book;
- cash;
- NAV;
- holdings;
- aportaciones;
- recomendaciones;
- fills;
- revisiones;
- scheduler.

Para cada una indicar:

- botón;
- endpoint;
- confirmación;
- preview;
- idempotencia;
- consecuencia;
- posibilidad de rollback;
- resultado visible;
- movement_id o identificador generado.

El rediseño no debe cambiar su semántica.

Las acciones económicas deben distinguirse claramente de:

- navegación;
- filtros;
- actualización;
- expansión de detalles;
- acciones informativas.

No añadir ejecución automática de órdenes Wio.

==================================================
I. RELACIÓN CON TRACK RECORD
==================================================

El Cockpit no debe duplicar toda la nueva pantalla Track record.

Debe mostrar únicamente un resumen accionable, por ejemplo:

- rendimiento principal;
- benchmark principal;
- diferencial;
- drawdown;
- fecha de actualización;
- estado de significancia o historia insuficiente;
- enlace claro a Track record.

Los gráficos completos de:

- universo equiponderado;
- diferencial;
- barras diarias;
- significancia;
- intervenciones;

permanecen en Track record.

Proponer exactamente qué información debe aparecer resumida en el Cockpit y
qué información debe permanecer exclusivamente en Track record.

==================================================
J. REGLAS DE IMPLEMENTACIÓN FUTURA
==================================================

La implementación posterior debe cumplir:

1. rama independiente feature/cockpit-v2;
2. feature flag COCKPIT_V2;
3. Cockpit anterior preservado;
4. funciones de cálculo existentes intactas;
5. endpoints existentes intactos salvo inserciones compatibles;
6. sin dependencias nuevas;
7. ES/EN completo;
8. tokens visuales existentes;
9. componentes nuevos en archivos nuevos cuando la arquitectura lo permita;
10. ninguna escritura económica durante pruebas visuales;
11. sin refactorizaciones no relacionadas;
12. sin cambios en Growth C1;
13. sin cambios en Track record salvo el enlace o resumen aprobado;
14. sin cambios en Gate 2F;
15. sin órdenes a Wio.

==================================================
K. PROPUESTA DE IMPLEMENTACIÓN
==================================================

El Checkpoint A debe proponer, pero no ejecutar:

COCKPIT_V2_NEW_FILES =
<lista>

COCKPIT_V2_INSERTION_POINTS =
<lista>

COCKPIT_V2_COMPONENTS =
<lista>

COCKPIT_V2_DATA_REUSE =
<endpoints y campos existentes>

COCKPIT_V2_NEW_ENDPOINTS_REQUIRED =
<lista/NONE>

COCKPIT_V2_EXISTING_CALCULATION_CHANGES_REQUIRED =
<lista/NONE>

COCKPIT_V2_FEATURE_FLAG_DESIGN =
<descripción>

COCKPIT_V2_COMMIT_PLAN =
<commits propuestos>

Si requiere modificar una función de cálculo existente:

- detenerse;
- explicar por qué;
- esperar aprobación.

==================================================
L. WIREFRAME DEL COCKPIT
==================================================

Entregar un wireframe textual para desktop y otro para viewport pequeño.

Formato sugerido:

┌────────────────────────────────────────────────────┐
│ ESTADO OPERATIVO · mercado · datos · actualización │
├────────────────────────────────────────────────────┤
│ ACCIONES REQUERIDAS                                │
├──────────────────────┬─────────────────────────────┤
│ NAV / CASH / INVERTIDO│ RIESGO / DRAWDOWN          │
├──────────────────────┴─────────────────────────────┤
│ RENDIMIENTO RESUMIDO · benchmark · diferencial     │
├────────────────────────────────────────────────────┤
│ PRÓXIMA REVISIÓN / PROCESOS PENDIENTES             │
├────────────────────────────────────────────────────┤
│ DETALLES TÉCNICOS [expandible]                     │
└────────────────────────────────────────────────────┘

No implementar el wireframe antes de que Omar lo apruebe.

==================================================
M. ENTREGA ADICIONAL DEL CHECKPOINT A
==================================================

Añadir a la entrega:

COCKPIT_FILES =
<lista>

COCKPIT_ENDPOINTS =
<lista>

COCKPIT_VISIBLE_SECTIONS =
<lista>

COCKPIT_ECONOMIC_ACTIONS =
<lista>

COCKPIT_DUPLICATED_INFORMATION =
<lista>

COCKPIT_SCATTERED_INFORMATION =
<lista>

COCKPIT_AMBIGUOUS_METRICS =
<lista>

COCKPIT_MISSING_CONTEXT =
<lista>

COCKPIT_QUESTIONS_ANSWERABLE =
<número/total>

COCKPIT_PROPOSED_INFORMATION_ARCHITECTURE =
<resumen>

COCKPIT_DESKTOP_WIREFRAME =
<wireframe>

COCKPIT_SMALL_VIEWPORT_WIREFRAME =
<wireframe>

COCKPIT_V2_NEW_FILES =
<lista>

COCKPIT_V2_EXISTING_FUNCTIONS_TO_MODIFY =
<lista/NONE>

COCKPIT_V2_NEW_DEPENDENCIES =
0

COCKPIT_V2_IMPLEMENTATION_BRANCH =
feature/cockpit-v2

COCKPIT_V2_STATUS =
AUDITED_PENDING_APPROVAL

No implementar Cockpit V2 todavía.

Detenerse junto con el Checkpoint A del Track record y esperar confirmación.