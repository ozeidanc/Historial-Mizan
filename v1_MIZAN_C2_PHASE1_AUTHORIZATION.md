MIZAN · MOTOR DE SELECCIÓN C2
AUTORIZACIÓN POSTERIOR A FASE 0

El archivo `spec-mizan-c2-seleccion.md` que precede a esta orden es normativo.

Sus secciones:

- PARTE A · Diagnóstico;
- PARTE B · Restricciones de arquitectura;

deben aplicarse íntegramente.

La Fase 0 queda aprobada.

BASELINE

BRANCH =
feature/c2-continuous-scoring

SOURCE_HEAD =
23978af

PRODUCTION_HEAD =
1d67c54

PRODUCTION_SCHEMA =
v31

C0_METHODOLOGY_HASH =
103b4256f26d9e94c7c64730cd9e04c2953119962d2e246b2aef9d8302854182

==================================================
1. DECISIÓN SOBRE LA RUTA
==================================================

Autorizar:

PRIMARY_ROUTE =
ROUTE_5_FORWARD_SHADOW_ONLY

Y, en paralelo:

DATA_PROVENANCE_ROUTE =
ROUTE_4_ARCHIVE_PIT_FROM_NOW_FORWARD

C2 puede construirse y ejecutarse en modo sombra.

C2 no puede promoverse mediante un backtest histórico contaminado.

El archivo PIT debe comenzar a acumular datos reproducibles desde su
despliegue en adelante.

No afirmar que este archivo repara las sesiones históricas anteriores.

==================================================
2. PRESERVACIÓN ABSOLUTA DE C0 Y C1
==================================================

Confirmo:

- no modificar C0;
- no modificar C1;
- no modificar sus archivos sellados;
- no modificar sus hashes;
- no cambiar su cadencia operativa bajo el mismo methodology_hash;
- no reescribir freezes históricos;
- no reinterpretar recomendaciones históricas.

Aunque la Fase 0 haya confirmado que el runtime de C0/C1 contradice la
frecuencia trimestral sellada, la corrección se implementará exclusivamente
en C2.

Entregar siempre:

C0_FILE_MODIFIED =
NO

C0_METHODOLOGY_HASH_UNCHANGED =
PASS

C1_FILE_MODIFIED =
NO

C1_RUNTIME_MODIFIED =
NO

==================================================
3. INCONSISTENCIA 11 FACTORES FRENTE A 15 CHECKS
==================================================

Antes de escribir el score continuo, resolver esta discrepancia:

- la especificación habla de 11 factores;
- el audit trail contiene 15 checks;
- la Fase 0 enumera más de 11 variables;
- algunas parecen gates, otras factores de ranking y otras información de
  display.

Crear un contrato exacto:

ACTIVE_BINARY_CHECKS =
<lista>

ELIGIBILITY_GATES =
<lista>

CONTINUOUS_RANKING_FACTORS =
<lista>

DISPLAY_ONLY_CHECKS =
<lista>

RISK_ONLY_CHECKS =
<lista>

Para cada check entregar:

CHECK_ID =
<id>

CURRENT_ROLE =
ELIGIBILITY_GATE /
BINARY_RANKING /
DISPLAY_ONLY /
RISK_ONLY

GREEN_RATE =
<porcentaje>

COVERAGE =
<porcentaje>

RAW_VALUE_AVAILABLE =
YES/NO

RAW_VALUE_SEMANTICS =
<descripción>

PIT_STATUS =
SAFE/UNSAFE/PARTIAL

C2_PROPOSED_ROLE =
<rol>

No implementar el compuesto hasta que:

- el número de factores esté cerrado;
- su semántica esté documentada;
- cada fórmula continua tenga un valor crudo real;
- los factores de display no entren accidentalmente en el ranking.

Si esta clasificación requiere una decisión económica de Omar, detenerse y
preguntar antes de implementar el score.

==================================================
4. FASE 1.0 · ARCHIVO PIT APPEND-ONLY
==================================================

Implementar primero el archivo point-in-time necesario para observación
forward.

Debe capturar por sesión:

- session/evidence date;
- cutoff;
- ticker;
- universo y elegibilidad;
- valor crudo de cada factor;
- fuente;
- acceptedDate;
- fetchedAt;
- source period;
- cobertura;
- missing reason;
- checks binarios;
- metodología;
- content hash;
- input hash.

Requisitos:

- append-only;
- idempotente;
- immutable después de sellar;
- sin sobrescribir sesiones;
- sin usar el cache mutable como única evidencia;
- sin modificar prospective_data_audit existente;
- sin reutilizar C0_SELECTION_FROZEN;
- sin modificar C0/C1;
- sin crear recomendaciones.

Si se amplía prospective_data_audit, utilizar un kind nuevo y explícito:

C2_FACTOR_INPUT_SNAPSHOT

No utilizar:

C0_SELECTION_FROZEN

para los nuevos snapshots.

Un retry con el mismo input debe devolver el registro existente.

Un retry con el mismo identificador y contenido diferente debe producir:

CONTENT_CONFLICT

==================================================
5. LIMITACIÓN DEL ARCHIVO PIT
==================================================

El archivo PIT comienza a ser válido desde el primer snapshot nuevo sellado.

Declarar:

PIT_ARCHIVE_EFFECTIVE_FROM =
<fecha>

No usarlo para afirmar que existen fundamentales PIT anteriores a esa fecha.

Los ocho freezes históricos disponibles permanecen:

POINT_IN_TIME_SAFE =
PARTIAL

No reconstruir artificialmente sus fundamentales.

==================================================
6. C2 EN MODO SOMBRA
==================================================

Crear:

GROWTH_C2_CONTINUOUS_SCORING

Archivo sellado nuevo:

crecimiento-v2-c2-sealed.mjs

C2 debe tener:

status =
SHADOW

executable =
FALSE

C2 puede:

- calcular;
- ordenar;
- registrar;
- mantener una cartera teórica;
- medir rotación;
- producir auditoría;
- comparar contra C0/C1;
- acumular retorno sombra.

C2 no puede:

- crear recomendaciones ejecutables;
- superseder recomendaciones C1;
- cambiar holdings;
- cambiar cash;
- cambiar NAV;
- crear movimientos;
- crear formularios Wio;
- enviar órdenes;
- convertirse automáticamente en activa.

Persistir resultados con:

kind =
C2_SELECTION_SHADOW

No reutilizar:

C0_SELECTION_FROZEN

==================================================
7. COMPUTED SELECTION Y ACTIVE SHADOW SELECTION
==================================================

C2 debe separar:

computed_selection

de:

active_shadow_selection

COMPUTED_SELECTION

- se calcula cada sesión;
- utiliza los inputs PIT disponibles;
- se registra siempre;
- permite observar el ranking y el churn potencial;
- nunca ejecuta.

ACTIVE_SHADOW_SELECTION

- representa la cartera teórica C2;
- no es la selección productiva;
- no afecta a C1;
- cambia únicamente en la cadencia C2 autorizada.

Cadencia inicial:

scheduled_composition_rebalance =
QUARTERLY

La selección activa sombra puede cambiar únicamente por:

A. rebalanceo trimestral programado;
B. hard risk exit definido y documentado;
C. override sombra explícito y auditado.

No inventar reglas de hard risk.

Si no existe todavía una regla aprobada:

hard_risk_exit =
NONE

==================================================
8. INICIALIZACIÓN DE LA SOMBRA
==================================================

En la primera sesión C2:

- calculated selection se genera normalmente;
- active shadow selection debe inicializarse de forma explícita;
- registrar la regla de inicialización;
- no afirmar que fue un rebalanceo trimestral histórico.

Propuesta inicial:

SHADOW_INITIALIZATION =
FIRST_VALID_C2_COMPUTED_SELECTION

Debe registrarse como evento de inicialización, no como retorno generado por
una decisión previa.

La medición de rendimiento comienza después de la primera sesión ejecutable
teórica definida por el contrato de timing.

==================================================
9. SCORE CONTINUO
==================================================

Después de cerrar el contrato de factores, implementar:

1. valor crudo continuo;
2. winsorización transversal;
3. percentil transversal;
4. missing neutral = 0.5;
5. media aritmética;
6. pesos iguales.

No optimizar pesos.

No convertir directamente:

green / amber / red

en un falso continuo.

El continuo debe partir de magnitudes económicas reales.

Cada factor debe definir:

- fórmula;
- dirección;
- unidad;
- fuente;
- missing policy;
- cobertura;
- winsorización;
- percentil;
- PIT status.

==================================================
10. DEFINICIÓN DETERMINISTA
==================================================

WINSORIZACIÓN

Usar percentiles 1 y 99 del universo elegible de la sesión.

Documentar y probar:

- método de interpolación;
- pocos datos;
- todos los valores iguales;
- un solo valor;
- valores duplicados;
- infinities;
- NaN.

PERCENTILE RANK

Debe ser determinista.

Para empates económicos, asignar el mismo percentile rank, utilizando el
método acordado y probado.

Ticker no debe cambiar el percentile económico.

El orden final puede usar:

1. continuous_score DESC;
2. revGrowthPct DESC;
3. ticker ASC únicamente como estabilidad final.

Reportar cuántas plazas quedan decididas por ticker después del continuo.

Objetivo:

ALPHABETICALLY_DETERMINED_SEATS =
0

No forzar artificialmente cero si existen empates económicos reales.

==================================================
11. GATE BINARIO Y DATOS AUSENTES
==================================================

La Fase 0 confirmó:

VARIABLE_DENOMINATOR_AFFECTS_ELIGIBILITY =
YES

Pero cambiar simultáneamente:

- score de ranking;
- gates de elegibilidad;
- cobertura;

dificultaría atribuir el efecto.

Por tanto, implementar y registrar en sombra dos variantes no ejecutables:

C2A =
CONTINUOUS_RANKING_WITH_LEGACY_ELIGIBILITY

C2B =
CONTINUOUS_RANKING_WITH_MINIMUM_COVERAGE

C2A permite aislar el efecto del ranking continuo.

C2B añade una política explícita de cobertura.

Ninguna es ejecutable.

No utilizar todavía un denominador fijo arbitrario de 15 sin cerrar el
contrato de checks.

Para C2B, proponer y configurar:

minimum_factor_coverage

pero no fijar silenciosamente el umbral.

Entregar un preview para varios umbrales razonables.

No eliminar automáticamente MELI, TXN u otros tickers sin una política
aprobada.

==================================================
12. FACTORES NO DISCRIMINANTES
==================================================

No retirar automáticamente todos los factores marcados por la Fase 0.

Distinguir:

A. GATES

Ejemplos posibles:

- rev_grow;
- mcap;
- coverage.

Pueden ser deliberadamente binarios y no pretender ordenar.

B. BAJA COBERTURA

Ejemplo:

- insider.

No debe entrar en el compuesto como si tuviera evidencia suficiente.

C. ESTABLES POR DISEÑO

Ejemplos posibles:

- debt;
- fcf;
- surprise.

Requieren explicación económica antes de retirarlos.

D. FACTORES DE RANKING

Solo estos forman el compuesto continuo.

Para la primera sombra, permitir comparar:

- compuesto completo aprobado;
- compuesto reducido diagnóstico.

Pero solo una variante puede ser la candidata C2 canónica después de
aprobación.

==================================================
13. HISTÉRESIS PURA
==================================================

Mantener:

seleccionar(universo, config, seleccionPrevia)

como función pura.

Prohibido acceder a:

- BD;
- red;
- estado global mutable;

desde seleccionar().

Contrato:

1. ordenar elegibles por score económico;
2. identificar incumbentes de seleccionPrevia;
3. retener incumbentes con rank <= exit_rank_buffer;
4. si hay más de 25 retenidos, conservar los 25 con mejor rank;
5. rellenar plazas libres con los mejores no incumbentes elegibles;
6. devolver como máximo 25;
7. utilizar revGrowthPct como primer desempate económico;
8. utilizar ticker únicamente como desempate estable final.

Configuración inicial de sombra:

entry_rank =
25

exit_rank_buffer =
30

Si existen 25 incumbentes dentro de rank 1–30:

- no entra un candidato nuevo únicamente por estar en rank 25;
- entra cuando se libera una plaza o cuando supera la regla económica
  explícita configurada.

No inventar una excepción de entrada.

Medir y reportar este efecto.

Cuando seleccionPrevia = null:

- no aplicar histéresis;
- elegir los 25 mejores del ranking C2;
- no afirmar que reproduce C0, porque C2 utiliza otro score;
- sí debe ser determinista y puro.

La exigencia de reproducir comportamiento legacy con seleccionPrevia=null se
aplica al selector C0 existente, que debe permanecer intacto.

==================================================
14. DRIFT BAND
==================================================

Configurar en C2 sombra:

driftBandRel =
0.20

Aplicar solo cuando:

- target_weight > 0;
- existe una posición sombra;
- no existe hard exit;
- no existe salida obligatoria;
- no existe target cero.

No bloquear:

- ELIMINAR;
- hard exits;
- target_weight = 0;
- posición no elegible;
- reglas de riesgo.

Mantener por separado:

minimum_executable_gross_notional =
2.00

La drift band determina si hace falta rebalancear por peso.

La materialidad de 2 USD determina si una operación calculada es ejecutable.

En modo sombra, “ejecutable” significa operación teórica válida, nunca una
recomendación real.

==================================================
15. CARTERA SOMBRA REPRODUCIBLE
==================================================

No limitar la sombra a una lista de tickers.

Definir:

- capital inicial teórico;
- holdings iniciales;
- fecha inicial;
- sesión de señal;
- sesión y precio de ejecución teóricos;
- comisiones;
- slippage;
- cash residual;
- dividendos;
- splits;
- corporate actions;
- delistings;
- missing prices;
- rebalances trimestrales;
- drift band;
- materialidad;
- valoración EOD;
- TWR;
- metodología hash;
- input hash;
- lineage.

Por defecto, no inventar costes.

Si se utiliza una política existente de costes simulados, documentarla.

Si no existe, entregar escenarios separados:

GROSS

y:

NET_UNDER_CONFIGURED_COST_ASSUMPTIONS

La sombra no debe beneficiarse de precios que no estaban disponibles en el
momento de la señal.

==================================================
16. BENCHMARKS DE VALIDACIÓN
==================================================

La sombra debe compararse, con la misma base y timing, contra:

- SPY;
- QQEW;
- universo elegible equiponderado.

El universo elegible equiponderado debe utilizar la elegibilidad disponible en
cada sesión y metodología documentada.

No comparar:

- C2 TWR;
- contra C0 money-weighted P&L.

Usar series homogéneas y neutrales a flujos.

==================================================
17. PIT Y VALIDACIÓN
==================================================

Declarar permanentemente:

HISTORICAL_C0_VS_C2_BACKTEST_VALID =
NO

hasta disponer de una reconstrucción PIT válida.

Puede existir un análisis contaminado para depuración, pero debe rotularse:

NON_CAUSAL_CONTAMINATED_DIAGNOSTIC

No puede utilizarse para:

- aprobar;
- promover;
- optimizar;
- elegir pesos;
- retirar factores;
- declarar alfa.

La evidencia principal será la sombra forward.

Horizonte inicial:

MINIMUM_SHADOW_OBSERVATION_SESSIONS =
60

Horizonte preferido:

3_TO_6_MONTHS

No promover C2 únicamente por superar 60 sesiones.

==================================================
18. TEST DORADO
==================================================

Crear un test que reproduzca desde los artefactos sellados disponibles:

- ranking C0;
- selection;
- ordering;
- hashes;
- methodology_hash.

Sesiones disponibles inicialmente:

8

El criterio original solicitaba al menos 10.

No reducir silenciosamente el criterio.

Entregar:

CURRENT_GOLDEN_SESSIONS =
8

TARGET_GOLDEN_SESSIONS =
10

El test puede comenzar con 8, pero la promoción permanece bloqueada hasta
tener al menos 10 sesiones reproducibles o una autorización expresa posterior.

No recalcular fundamentales históricos usando el cache actual.

==================================================
19. FEATURE FLAGS
==================================================

C2 debe estar detrás de flags independientes:

GROWTH_C2_PIT_ARCHIVE
GROWTH_C2_SHADOW_COMPUTE
GROWTH_C2_SHADOW_PORTFOLIO

Defaults:

false

La activación debe ser independiente.

Orden de activación:

1. PIT archive;
2. shadow compute;
3. shadow portfolio.

No crear:

GROWTH_C2_EXECUTABLE

en esta fase.

No debe existir un camino accidental de C2 hacia recomendaciones reales.

==================================================
20. COMMITS
==================================================

Un cambio por commit.

Orden mínimo:

1.
docs(growth): add normative C2 selection specification

2.
feat(growth): archive point-in-time C2 factor inputs

3.
feat(growth): add sealed C2 shadow methodology

4.
feat(growth): add continuous cross-sectional scoring

5.
feat(growth): add economic tie-breaking for C2

6.
feat(growth): add pure C2 selection hysteresis

7.
feat(growth): add C2 shadow sizing drift band

8.
feat(growth): add reproducible C2 shadow portfolio

9.
test(growth): add C0 golden selection regression

Si se implementan C2A y C2B:

- configurarlas dentro del contrato sombra;
- no mezclar sus resultados;
- conservar lineage y hash diferenciados.

No hacer push.
No crear tag.

==================================================
21. CHECKPOINT ANTES DE ESCRITURAS PRODUCTIVAS
==================================================

Antes de desplegar incluso el archivo PIT, entregar:

NORMATIVE_SPEC_PRESENT =
PASS/FAIL

FACTOR_CONTRACT_COMPLETE =
PASS/FAIL

ACTIVE_BINARY_CHECK_COUNT =
<número>

CONTINUOUS_RANKING_FACTOR_COUNT =
<número>

DISPLAY_ONLY_CHECK_COUNT =
<número>

PIT_ARCHIVE_SCHEMA =
<descripción>

PIT_ARCHIVE_IDEMPOTENCY =
PASS/FAIL

C2_SEALED_FILE =
<ruta>

C2_METHODOLOGY_HASH =
<hash>

C2_STATUS =
SHADOW

C2_EXECUTABLE =
FALSE

C0_GOLDEN_TEST =
PASS/FAIL

CURRENT_GOLDEN_SESSIONS =
8

C0_HASH_UNCHANGED =
PASS/FAIL

C1_UNCHANGED =
PASS/FAIL

C2A_PREVIEW =
<resumen>

C2B_COVERAGE_PREVIEW =
<resumen>

SHADOW_PORTFOLIO_CONTRACT =
PASS/FAIL

NEW_DEPENDENCIES =
0

PRODUCTION_WRITES =
0

Detenerse en este checkpoint.

No desplegar ni activar flags hasta recibir autorización expresa de Omar.