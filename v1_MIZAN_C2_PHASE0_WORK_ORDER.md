# Mizan · Motor de selección C2
## Orden de ejecución · Fase 0 read-only

Lee íntegramente el archivo adjunto:

`spec-mizan-c2-seleccion.md`

El archivo completo forma parte normativa de esta orden.

Las secciones:

- `PARTE A · Diagnóstico`
- `PARTE B · Restricciones de arquitectura`

deben aplicarse literalmente, sin resumirse, reinterpretarse ni omitirse.

Las fases deben ejecutarse estrictamente en este orden:

1. FASE 0;
2. checkpoint y autorización expresa de Omar;
3. FASE 1;
4. FASE 2;
5. autorización expresa de promoción;
6. FASE 3.

En esta sesión debes ejecutar exclusivamente la FASE 0.

No escribas código de producción.
No crees todavía la metodología C2.
No modifiques C0 ni C1.
No despliegues.
No generes recomendaciones.
No realices escrituras económicas.

==================================================
0. BASELINE Y RAMA
==================================================

Baseline esperado:

PRODUCTION_HEAD =
1d67c54

LOCAL_HEAD esperado =
23978af

SCHEMA =
v31

La diferencia conocida es:

23978af =
1d67c54 + commit de tests y auditoría, sin cambios productivos.

Crear o utilizar:

feature/c2-continuous-scoring

La rama debe partir del HEAD local autorizado después de verificar el grafo.

No trabajar sobre:

- main;
- feature/track-record-benchmarks;
- feature/cockpit-v2.

Antes de continuar:

- comprobar branch;
- comprobar HEAD;
- comprobar git status;
- inventariar untracked;
- confirmar schema;
- confirmar una sola raíz Git;
- confirmar que el archivo sellado C0 está intacto;
- registrar el methodology_hash actual;
- ejecutar integrity_check y foreign_key_check si se accede a una copia de BD.

No resetear ni descartar trabajo.

==================================================
1. RESTRICCIÓN ABSOLUTA DE LA FASE 0
==================================================

La Fase 0 es read-only.

Permitido:

- leer código;
- consultar datos;
- ejecutar SELECT;
- leer audit trails;
- analizar snapshots persistidos;
- ejecutar scripts temporales fuera de producción;
- crear artefactos de informe fuera del código productivo.

Prohibido:

- modificar código de producción;
- modificar configuración;
- modificar archivos sellados;
- crear tablas;
- migrar schema;
- insertar auditorías;
- recalcular recomendaciones productivas;
- recongelar selecciones;
- modificar el scheduler;
- crear recomendaciones C2;
- crear commits funcionales.

Si para obtener una medición hace falta modificar producción, detenerse y
explicar el bloqueo.

==================================================
2. CUESTIONAMIENTO OBLIGATORIO DEL DIAGNÓSTICO
==================================================

No asumir que las cinco conclusiones del diagnóstico son correctas.

Para cada una entregar:

DIAGNOSIS_ID =
A1/A2/A3/A4/A5

CLASSIFICATION =
CONFIRMED /
PARTIALLY_CONFIRMED /
NOT_CONFIRMED /
NOT_MEASURABLE

EVIDENCE =
<datos reproducibles>

ALTERNATIVE_EXPLANATIONS =
<lista>

CONFIDENCE =
HIGH/MEDIUM/LOW

Específicamente:

A1:
Demostrar cuántas plazas fueron decididas realmente por ticker_asc.

A2:
Distinguir el efecto de denominadores distintos en:
- el ranking;
- el gate binario;
- la elegibilidad final.

A3:
Demostrar si el freeze nocturno sustituye la composición activa y si fue la
causa concreta de la salida de PANW.

A4:
No concluir “alfa cero” únicamente porque se seleccionen 25 de unas 30.
Medir el grado real de selectividad y su límite.

A5:
Clasificar el gate sectorial como:
- restricción explícita de política;
- resultado emergente de selección;
- o combinación de ambos.

==================================================
3. FUENTE HISTÓRICA DE LAS MEDICIONES
==================================================

Las mediciones históricas deben utilizar, por orden de preferencia:

1. snapshots sellados por sesión;
2. prospective_data_audit;
3. C0_SELECTION_FROZEN persistido;
4. inputs archivados con timestamp y hash;
5. otras fuentes históricas reproducibles.

No recalcular una sesión pasada usando datos fundamentales actuales.

Para cada métrica indicar:

SOURCE_TABLE_OR_ARTIFACT =
<fuente>

AS_OF_SEMANTICS =
<semántica temporal>

POINT_IN_TIME_SAFE =
YES/NO/PARTIAL

RECOMPUTED_WITH_CURRENT_DATA =
YES/NO

Si no existen inputs históricos suficientes:

- marcar la medición como no reproducible;
- no completar valores inventados;
- explicar qué dato falta;
- no presentar el resultado como histórico PIT.

==================================================
4. FASE 0.1 · EMBUDO DE ELEGIBILIDAD
==================================================

Analizar las últimas 20 sesiones disponibles o todas las disponibles si hay
menos de 20.

Entregar:

SESSIONS_REQUESTED =
20

SESSIONS_AVAILABLE =
<número>

Para cada sesión:

- universo curado;
- tras datos_sospechosos;
- tras gate sectorial;
- tras rev_grow;
- tras revGrowthPct;
- tras score binario;
- seleccionados;
- elegibles/25;
- seleccionados/elegibles;
- diferencia de score entre ranks 25 y 26.

Calcular:

- mínimo;
- mediana;
- máximo;
- percentiles relevantes.

La regla:

median(elegibles / 25) < 1.5

debe clasificarse como:

PRODUCT_DECISION_HEURISTIC

No como prueba matemática de ausencia de alfa.

Comparar cuando sea posible:

- top 25;
- todos los elegibles equiponderados.

No usar esa comparación como evidencia de rendimiento si no es PIT.

==================================================
5. FASE 0.2 · EMPATES Y TICKER_ASC
==================================================

Para cada sesión:

- grupos con `(greens,total)` idénticos;
- tamaño de cada grupo;
- grupos que cruzan la frontera 25/26;
- plazas elegidas dentro de un empate;
- plazas cuya inclusión dependió de ticker_asc;
- plazas que habrían cambiado con otro desempate determinista.

Definir:

ALPHABETICALLY_DETERMINED_SEAT

como una plaza para la que:

- dos o más candidatos tienen iguales claves económicas implementadas;
- la frontera de selección atraviesa el grupo;
- ticker_asc decide quién queda dentro.

Entregar:

ALPHABETICALLY_DETERMINED_SEATS_BY_SESSION =
<serie>

MEDIAN_ALPHABETICALLY_DETERMINED_SEATS =
<número>

PORTFOLIO_TICKERS_A_TO_C =
<número>

UNIVERSE_TICKERS_A_TO_C =
<número>

EXPECTED_A_TO_C_UNDER_RANDOM_SELECTION =
<valor>

No inferir sesgo solo por observar AAPL o ADI en posiciones altas.

==================================================
6. FASE 0.3 · CAPACIDAD DISCRIMINANTE
==================================================

Para cada factor activo:

- verdes;
- ámbar;
- rojos;
- na;
- tasa de verdes sobre evaluados;
- tasa de cobertura;
- entropía o medida descriptiva equivalente;
- variación entre sesiones.

Marcar:

POTENTIALLY_NON_DISCRIMINANT

si:

green_rate > 90 %

o:

green_rate < 10 %

Pero no autorizar automáticamente su retirada.

La posible retirada de un factor requiere:

- resultado de Fase 0;
- explicación económica;
- aprobación expresa;
- configuración versionada en C2.

Distinguir:

- factor no discriminante;
- factor con baja cobertura;
- factor estable por diseño;
- gate de riesgo que no pretende ordenar.

==================================================
7. FASE 0.4 · ESTABILIDAD
==================================================

Reconstruir por sesión:

- rank;
- score binario;
- greens;
- total;
- elegibilidad;
- selección calculada;
- selección activa, si son conceptos distintos.

Medir:

- cruces 25→26;
- cruces 26→25;
- cruces 25/30;
- sustituciones;
- permanencia;
- solapamiento entre sesiones;
- tickers oscilantes.

Auditar específicamente PANW:

- inputs;
- score;
- rank;
- selección anterior;
- selección nueva;
- freeze;
- recomendación;
- ejecución;
- salida;
- pérdida realizada.

Declarar:

PANW_EXIT_CAUSED_BY_NIGHTLY_REFREEZE =
PASS/FAIL/NOT_PROVABLE

No atribuir causalidad si solo existe correlación temporal.

==================================================
8. FASE 0.5 · CORRELACIÓN
==================================================

Construir una matriz para los checks activos.

Documentar:

- método de correlación;
- codificación g/a/r/na;
- tratamiento de missing;
- universo usado;
- fechas;
- número de observaciones.

No convertir automáticamente:

g/a/r

en números arbitrarios sin justificarlo.

Para checks binarios puede utilizarse una medida compatible, como phi/Pearson
sobre variables binarias, siempre que los missing se manejen explícitamente.

Analizar especialmente:

- pe_hist;
- pe_sect;
- below_tgt;
- eps_rev;
- surprise;
- ma200.

La correlación negativa entre valor y momentum no demuestra por sí sola que el
compuesto sea incorrecto. Entregar interpretación y alternativas.

==================================================
9. CONTRADICCIÓN DEL DENOMINADOR
==================================================

La especificación propone:

- missing = 0.5 en el score continuo;
- mantener inicialmente el gate binario legacy.

Esto resuelve denominadores distintos en el ranking continuo, pero no
necesariamente en el gate de elegibilidad.

Medir y entregar:

VARIABLE_DENOMINATOR_AFFECTS_RANKING =
YES/NO

VARIABLE_DENOMINATOR_AFFECTS_ELIGIBILITY =
YES/NO

TICKERS_WHOSE_ELIGIBILITY_CHANGES_UNDER_FIXED_DENOMINATOR =
<lista>

TICKERS_WHOSE_ELIGIBILITY_CHANGES_UNDER_MINIMUM_COVERAGE =
<lista>

Proponer, sin implementar, alternativas:

A. LEGACY_GATE_TEMPORARILY_PRESERVED

Útil para atribuir el efecto del score continuo, pero conserva el problema en
el gate.

B. MINIMUM_FACTOR_COVERAGE

Exigir una cobertura mínima antes del gate.

C. FIXED_DENOMINATOR_WITH_NEUTRAL_MISSING

Usar tratamiento neutral también en el gate.

D. REMOVE_BINARY_SCORE_GATE_IN_LATER_C2_VERSION

Mantener gates económicos y utilizar el continuo.

No decidir sin aprobación.

==================================================
10. CADENCIA Y C0/C1
==================================================

Verificar el contrato sellado de frecuencia trimestral.

Distinguir:

- computed_selection;
- active_selection;
- daily sizing;
- recommendation generation;
- quarterly composition rebalance;
- hard risk override;
- manual override.

No modificar C0/C1.

Aunque se confirme que la ejecución actual contradice el sello:

- documentar la contradicción;
- no cambiar silenciosamente el runtime bajo el mismo methodology_hash;
- implementar la corrección únicamente en C2, salvo una autorización futura
  independiente.

Entregar:

C0_C1_RUNTIME_MATCHES_SEALED_FREQUENCY =
PASS/FAIL

NIGHTLY_FREEZE_CHANGES_ACTIVE_COMPOSITION =
YES/NO

DAILY_RECOMMENDATIONS_REQUIRE_DAILY_COMPOSITION_CHANGE =
YES/NO

==================================================
11. VIABILIDAD DEL SCORE CONTINUO
==================================================

Sin implementarlo todavía, inventariar para cada uno de los 11 factores:

- valor crudo disponible;
- fuente;
- dirección;
- unidad;
- cobertura;
- disponibilidad histórica;
- acceptedDate/cutoff;
- posibilidad de cálculo PIT;
- fórmula continua propuesta.

Entregar:

FACTOR_ID =
<id>

RAW_VALUE_AVAILABLE =
YES/NO

RAW_SOURCE =
<fuente>

DIRECTION =
HIGHER_IS_BETTER/LOWER_IS_BETTER

COVERAGE =
<porcentaje>

PIT_STATUS =
SAFE/UNSAFE/PARTIAL/UNKNOWN

PROPOSED_CONTINUOUS_FORMULA =
<fórmula>

No crear una fórmula continua si el dato fuente no existe o su semántica no
está clara.

==================================================
12. DEFINICIONES PENDIENTES PARA FASE 1
==================================================

Proponer definiciones deterministas, sin implementarlas:

A. WINSORIZACIÓN

- método de percentil;
- interpolación;
- comportamiento con pocos valores;
- comportamiento con todos los valores iguales.

B. PERCENTILE RANK

- tratamiento de empates;
- estabilidad;
- orden;
- neutral exacto 0.5 para missing.

C. HISTÉRESIS

Proponer un algoritmo que siempre produzca como máximo 25 seleccionados.

Contrato sugerido:

1. ordenar todos los elegibles económicamente;
2. retener incumbentes con rank ≤ exit_rank_buffer;
3. si hay más de 25 retenidos, conservar los 25 de mejor rank;
4. rellenar plazas libres con no incumbentes de mejor rank que cumplan la
   regla de entrada;
5. aplicar desempate económico;
6. ticker_asc solo como desempate final estable.

Aclarar cómo entra un candidato nuevo si existen 25 incumbentes entre los
ranks 1 y 30.

D. DRIFT BAND

Aplicar únicamente cuando:

- target_weight > 0;
- existe posición;
- no hay hard exit;
- no hay regla de riesgo.

No permitir que driftBandRel bloquee:

- ELIMINAR;
- una salida obligatoria;
- una posición con target_weight = 0.

==================================================
13. POINT-IN-TIME
==================================================

Auditar qué factores pueden reconstruirse con:

acceptedDate ≤ cutoff

Entregar:

PIT_FACTORS_SAFE =
<lista>

PIT_FACTORS_UNSAFE =
<lista>

PIT_FACTORS_PARTIAL =
<lista>

HISTORICAL_FUNDAMENTAL_ARCHIVE_AVAILABLE =
YES/NO/PARTIAL

PIT_BACKTEST_CURRENTLY_VALID =
YES/NO

No ejecutar ni presentar como evidencia un backtest C0 vs C2 contaminado.

Puede ejecutarse únicamente para depuración mecánica si queda rotulado:

NON_CAUSAL_CONTAMINATED_DIAGNOSTIC

y no se utiliza para promover C2.

==================================================
14. DISEÑO DE SOMBRA FORWARD
==================================================

Proponer cómo medir C2 durante 3–6 meses si el PIT no es viable.

Definir:

- frecuencia de cálculo;
- selección calculada;
- selección activa teórica;
- precios de entrada/salida;
- timing de ejecución simulado;
- costes;
- comisiones;
- slippage;
- cash residual;
- tratamiento de corporate actions;
- benchmark;
- auditoría;
- hash;
- lineage;
- ausencia total de recomendaciones ejecutables.

La cartera sombra debe tener reglas reproducibles.

No basta con registrar una lista diaria de tickers.

==================================================
15. TEST DORADO · PREPARACIÓN
==================================================

Sin modificar código productivo, determinar si existen al menos 10 sesiones
históricas con inputs suficientes para reproducir C0 byte a byte.

Entregar:

GOLDEN_SESSIONS_AVAILABLE =
<número>

GOLDEN_SESSION_IDS =
<lista>

EXPECTED_AUDIT_ARTIFACTS =
<lista>

C0_METHODOLOGY_HASH =
<hash>

BYTE_IDENTICAL_REPRODUCTION_CURRENTLY_POSSIBLE =
YES/NO/PARTIAL

Si no es posible, explicar exactamente qué input histórico falta.

No debilitar el criterio byte-idéntico.

==================================================
16. CHECKPOINT DE DECISIÓN
==================================================

Después de completar la Fase 0, recomendar una de estas rutas:

ROUTE_1 =
CONTINUE_C2_AS_SPECIFIED

ROUTE_2 =
CONTINUE_C2_WITH_GATE_COVERAGE_CHANGE

ROUTE_3 =
PRIORITIZE_UNIVERSE_EXPANSION_LATER_BUT_BUILD_STABILITY_FIRST

ROUTE_4 =
STOP_C2_AND_FIX_DATA_PROVENANCE_FIRST

ROUTE_5 =
FORWARD_SHADOW_ONLY_UNTIL_PIT_HISTORY_EXISTS

La recomendación debe derivarse de las mediciones.

No comenzar Fase 1.

==================================================
17. ENTREGA OBLIGATORIA
==================================================

BRANCH =
feature/c2-continuous-scoring

SOURCE_HEAD =
<hash>

PRODUCTION_HEAD =
1d67c54

PRODUCTION_SCHEMA =
v31

FILES_MODIFIED =
0

PRODUCTION_WRITES =
0

C0_FILE_MODIFIED =
NO

C0_METHODOLOGY_HASH_BEFORE =
<hash>

C0_METHODOLOGY_HASH_AFTER =
<mismo hash>

SESSIONS_ANALYZED =
<número>

DIAGNOSIS_1_SCORE_RESOLUTION =
CONFIRMED/PARTIAL/NOT_CONFIRMED

DIAGNOSIS_2_VARIABLE_DENOMINATORS =
CONFIRMED/PARTIAL/NOT_CONFIRMED

DIAGNOSIS_3_NO_HYSTERESIS =
CONFIRMED/PARTIAL/NOT_CONFIRMED

DIAGNOSIS_4_LOW_SELECTIVITY =
CONFIRMED/PARTIAL/NOT_CONFIRMED

DIAGNOSIS_5_SECTOR_CONCENTRATION =
POLICY_IMPOSED/SELECTION_EMERGENT/MIXED

MEDIAN_ELIGIBLES =
<número>

MEDIAN_ELIGIBLES_DIVIDED_BY_25 =
<valor>

MEDIAN_SELECTED_SHARE_OF_ELIGIBLES =
<porcentaje>

MEDIAN_ALPHABETICALLY_DETERMINED_SEATS =
<número>

MAX_ALPHABETICALLY_DETERMINED_SEATS =
<número>

NON_DISCRIMINANT_FACTORS =
<lista>

VARIABLE_DENOMINATOR_AFFECTS_ELIGIBILITY =
YES/NO

NIGHTLY_FREEZE_CHANGES_ACTIVE_COMPOSITION =
YES/NO

PANW_EXIT_CAUSED_BY_NIGHTLY_REFREEZE =
PASS/FAIL/NOT_PROVABLE

PIT_BACKTEST_CURRENTLY_VALID =
YES/NO

GOLDEN_SESSIONS_AVAILABLE =
<número>

BYTE_IDENTICAL_REPRODUCTION_CURRENTLY_POSSIBLE =
YES/NO/PARTIAL

RECOMMENDED_ROUTE =
<ROUTE_1..ROUTE_5>

PROPOSED_CHANGES_TO_ORIGINAL_SPEC =
<lista>

RISKS_FOUND_NOT_TOUCHED =
<lista/NONE>

EXACT_QUESTIONS_REQUIRING_APPROVAL =
<lista/NONE>

PHASE_0_STATUS =
COMPLETE

Detenerse después de esta entrega.

No comenzar Fase 1 hasta recibir autorización expresa de Omar.