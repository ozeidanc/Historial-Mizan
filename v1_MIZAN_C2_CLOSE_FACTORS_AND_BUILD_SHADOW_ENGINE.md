CHECKPOINT PIT CERTIFICADO APROBADO

PIT_CERTIFIED_COLLECTION =
COMPLETE

PIT_COMPLETE_SYMBOLS =
123

PIT_FAILED_SYMBOLS =
0

C0_C1_UNCHANGED =
PASS

COMPLIANT_ENGINE_STATUS =
READY_NOT_ACTIVE

No activar motores.
No desplegar.
No modificar C0/C1.
No refrescar la caché productiva.

==================================================
1. CUTOFF PIT EXACTO
==================================================

Persistir y entregar el cutoff certificado exacto:

PIT_FORWARD_EFFECTIVE_FROM =
<ISO-8601 timestamp con zona horaria>

No utilizar únicamente:

2026-07-28

El cutoff debe permitir demostrar qué información estaba disponible en el
momento de la captura.

Confirmar:

- inicio del fetch;
- final del fetch;
- cutoff lógico;
- zona horaria;
- price_asof;
- observed_at;
- acceptedDate;
- existencia o ausencia de publicaciones durante la captura.

Si la captura contiene datos con disponibilidades posteriores al cutoff
lógico, no presentarla como un único snapshot temporalmente coherente.

==================================================
2. PE_HIST
==================================================

Definición propuesta:

CURRENT_PE =

adjusted_session_price
/
current_ttm_diluted_eps

HISTORICAL_5Y_MEDIAN_PE =

mediana de observaciones mensuales válidas de trailing PE durante los 60
meses anteriores al cutoff.

PE_HIST =

(historical_5y_median_pe - current_pe)
/
historical_5y_median_pe

DIRECTION =
HIGHER_IS_BETTER

Cada observación histórica mensual debe utilizar:

- precio ajustado disponible en la fecha observada;
- TTM diluted EPS que podía conocerse en esa fecha;
- filings con acceptedDate <= observation_cutoff;
- corporate actions correctamente ajustadas;
- P/E positivo y finito.

No utilizar el EPS más reciente para recalcular retrospectivamente los cinco
años.

No utilizar cinco observaciones anuales de earnings yield como sustituto
silencioso.

TARGET_MONTHLY_OBSERVATIONS =
60

MINIMUM_VALID_MONTHLY_OBSERVATIONS =
24

Si existen menos de 24 observaciones PIT-certificables:

FACTOR_STATE =
UNDEFINED_REFERENCE

PERCENTILE =
0.5

FLAG =
INSUFFICIENT_PIT_HISTORICAL_PE

Si no es posible reconstruir las observaciones sin look-ahead:

PE_HIST_STATUS =
FORWARD_HISTORY_REQUIRED

No marcarlo READY basándose en historia contaminada.

==================================================
3. PE_SECT
==================================================

Definición:

PE_SECT =

(sector_median_current_pe - current_pe)
/
sector_median_current_pe

DIRECTION =
HIGHER_IS_BETTER

La referencia sectorial se calcula dentro del universo elegible de cada
variante:

C2_0_SECTOR_REFERENCE_UNIVERSE =
C2_0_ELIGIBLE_CURATED_UNIVERSE

C2_1_SECTOR_REFERENCE_UNIVERSE =
C2_1_ELIGIBLE_FULL_ROSTER

No utilizar la referencia C2.0 para C2.1.

Incluir únicamente miembros con:

- sector conocido;
- current PE positivo;
- current PE finito;
- precio certificado al cutoff;
- TTM EPS certificado al cutoff.

MINIMUM_VALID_SECTOR_PE_OBSERVATIONS =
5

Si un sector tiene menos de cinco observaciones válidas:

FACTOR_STATE =
UNDEFINED_REFERENCE

PERCENTILE =
0.5

FLAG =
INSUFFICIENT_SECTOR_SAMPLE

Persistir:

- sector;
- universe_variant;
- número de observaciones;
- mediana;
- miembros o hash determinista de miembros;
- cutoff;
- fórmula y versión.

Reportar sensibilidad diagnóstica para mínimos:

5
10

pero mantener 5 como propuesta canónica inicial.

==================================================
4. SURPRISE
==================================================

Variante canónica:

SURPRISE_OBSERVATIONS_REQUIRED =
4

Por cada publicación:

SURPRISE_PCT =

(actual_eps - estimated_eps)
/
abs(estimated_eps)

solo si:

estimated_eps != 0

y ambos valores son finitos y comparables.

Si estimated_eps = 0:

OBSERVATION_STATE =
UNDEFINED_PERCENTAGE_SURPRISE

La observación:

- no se convierte en Infinity;
- no recibe peor percentil;
- no se incluye en la media;
- conserva actual_eps y estimated_eps como evidencia cruda.

Usar las cuatro publicaciones comparables válidas más recientes disponibles
antes del cutoff.

Si existen menos de cuatro:

FACTOR_STATE =
PARTIAL_HISTORY

PERCENTILE =
0.5

FLAG =
SURPRISE_VALID_HISTORY_BELOW_4

Los valores porcentuales negativos válidos se conservan como señal adversa.

Entregar:

ESTIMATE_ZERO_CASES =
<número>

SURPRISE_BELOW_4_CASES =
<número>

SURPRISE_COVERAGE =
<porcentaje>

==================================================
5. ESTADO DE LOS FACTORES
==================================================

Después de aplicar estas reglas, entregar:

PE_HIST_STATUS =
READY /
FORWARD_HISTORY_REQUIRED /
PENDING_APPROVAL

PE_SECT_STATUS =
READY /
PENDING_APPROVAL

SURPRISE_STATUS =
READY /
PENDING_APPROVAL

No declarar:

FACTOR_FORMULA_CONTRACT =
PASS

hasta que los 11 factores tengan:

- fórmula;
- componentes fuente;
- dirección;
- temporalidad;
- política missing;
- política adversa;
- política no-finita;
- formula_version;
- tests.

==================================================
6. IMPLEMENTACIÓN DEL MOTOR GENÉRICO
==================================================

Autorizo construir mientras se verifica lo anterior, exclusivamente en LAB:

- winsorización transversal P1/P99;
- percentile ranks;
- inversión LOWER_IS_BETTER;
- missing neutral 0.5;
- cobertura mínima 9/11;
- denominador fijo 11;
- media con pesos iguales;
- top 25 determinista;
- histéresis pura mediante seleccionPrevia;
- audit output;
- methodology output schema.

Implementar mediante fixtures sintéticos y componentes inyectados.

No ejecutar todavía un C2.0 canónico con factores reales si alguno de los 11
permanece bloqueado.

Estado permitido:

C2_SCORING_ENGINE =
IMPLEMENTED_NOT_SEALED

C2_0_METHODOLOGY_OUTPUT =
NOT_GENERATED

==================================================
7. REGLAS DE PERCENTILES
==================================================

Sellar mediante tests:

WINSORIZATION =
P1_P99

PERCENTILE_RANGE =
[0,1]

MISSING_PERCENTILE =
0.5

Definir expresamente:

- interpolación de P1 y P99;
- método de rank para empates;
- un único valor;
- todos los valores iguales;
- dos valores;
- valores winsorizados iguales;
- dirección invertida;
- determinismo independiente del orden de entrada.

En igualdad económica, el ticker puede utilizarse únicamente como orden final
determinista.

Reportar:

ALPHABETICALLY_DETERMINED_FRONTIER_SEATS =
0

No afirmar que nunca existirán empates económicos.

==================================================
8. HISTÉRESIS
==================================================

Implementar como función pura.

Entrada:

- ranking completo;
- config;
- seleccionPrevia.

Reglas:

1. Con seleccionPrevia = null:
   seleccionar los mejores 25 del ranking C2.

2. Con selección previa:
   retener incumbentes con rank <= exit_rank_buffer.

3. Si hay más de 25 incumbentes retenidos:
   conservar los 25 incumbentes con mejor rank.

4. Si hay menos de 25:
   completar con los mejores no incumbentes.

5. Devolver exactamente 25 si existen al menos 25 elegibles.

Config inicial:

ENTRY_RANK_MAX =
25

EXIT_RANK_BUFFER =
30

No implementar HARD_RISK_EXIT.

HARD_RISK_EXIT =
NONE

La revisión diaria sombra no cambia la composición productiva.

==================================================
9. TESTS
==================================================

Probar:

- PE histórico sin look-ahead;
- PE histórico con menos de 24 observaciones;
- PE sectorial con 4, 5 y 10 miembros;
- hash de miembros sectoriales determinista;
- estimate EPS igual a cero;
- menos de cuatro sorpresas válidas;
- sorpresa negativa conservada;
- percentiles independientes del orden de entrada;
- todos los valores iguales;
- winsorización P1/P99;
- missing 0.5;
- adverso no neutralizado;
- 9/11 elegible;
- 8/11 no elegible;
- denominador fijo 11;
- histéresis con null;
- histéresis con incumbentes 26–30;
- más de 25 incumbentes retenidos;
- menos de 25 incumbentes;
- exactamente 25 resultados;
- cero escrituras productivas.

==================================================
10. COMMITS
==================================================

Crear commits separados:

1.
docs(growth): close remaining C2 factor contracts

2.
feat(growth): add pure continuous scoring primitives

3.
feat(growth): add pure C2 selection hysteresis

4.
test(growth): verify continuous scoring determinism

No generar todavía una sesión C2.0 canónica si algún factor permanece
bloqueado.

No hacer push.
No crear tag.

==================================================
11. CHECKPOINT
==================================================

Entregar:

PIT_FORWARD_EFFECTIVE_FROM =
<timestamp ISO-8601>

FACTOR_FORMULA_CONTRACT =
PASS/PENDING

READY_FACTORS =
<lista>

BLOCKED_FACTORS =
<lista>

PE_HIST_STATUS =
<estado>

PE_SECT_STATUS =
<estado>

SURPRISE_STATUS =
<estado>

C2_SCORING_ENGINE =
IMPLEMENTED_NOT_SEALED/NOT_STARTED

C2_HYSTERESIS =
IMPLEMENTED_NOT_ACTIVE/NOT_STARTED

PERCENTILE_CONTRACT =
PASS/FAIL

HYSTERESIS_CONTRACT =
PASS/FAIL

C2_0_METHODOLOGY_HASH =
NOT_YET

C2_0_METHODOLOGY_OUTPUT =
NOT_GENERATED

COMPLIANT_ENGINE_STATUS =
READY_NOT_ACTIVE

C0_C1_UNCHANGED =
PASS/FAIL

LEGACY_GOLDEN_HASH_UNCHANGED =
PASS/FAIL

NEW_TEST_FAILURES =
0

PRODUCTION_DEPLOY =
NO

PRODUCTION_WRITES =
0

PRODUCTION_RECOMMENDATIONS =
0

BROKER_ORDERS =
0

Detenerse después del checkpoint.

No sellar C2.0 hasta que los 11 factores estén READY.
No desplegar.
No activar ningún motor.