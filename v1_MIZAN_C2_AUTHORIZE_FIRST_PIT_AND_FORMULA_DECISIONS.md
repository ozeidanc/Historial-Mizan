CHECKPOINT APROBADO

Rama:

feature/c2-continuous-scoring

Producción permanece intacta:

PRODUCTION_HEAD =
1d67c54

PRODUCTION_SCHEMA =
v31

No implementar todavía el score continuo.
No activar ningún motor.
No desplegar.
No modificar la caché productiva.

==================================================
1. DECISIÓN SOBRE EPS_REV
==================================================

La semántica existente se preserva y se renombra correctamente:

OLD_FACTOR_ID =
eps_rev

CANONICAL_C2_FACTOR_ID =
target_rev_3m

ECONOMIC_MEANING =
revisión porcentual del precio objetivo de consenso entre la referencia
trimestral y la referencia mensual almacenadas por la fuente

RAW_FORMULA =

(lastMonthAvgPriceTarget - lastQuarterAvgPriceTarget)
/
lastQuarterAvgPriceTarget

DIRECTION =
HIGHER_IS_BETTER

TEMPORAL_CLASS =
ANALYST_ESTIMATE_OR_TARGET

No llamarlo revisión de EPS.

No cambiar silenciosamente la fuente o la semántica.

Si:

lastQuarterAvgPriceTarget <= 0

o falta cualquiera de los dos valores:

FACTOR_STATE =
TRUE_MISSING_OR_UNDEFINED

PERCENTILE =
0.5

Registrar:

- observed_at;
- fetched_at;
- provider_asof si existe;
- número de analistas si existe;
- valores mensual y trimestral originales;
- source payload lineage.

No inventar campos ausentes.

==================================================
2. EPS REVISION VERDADERA · HISTORIA FUTURA
==================================================

Preparar captura PIT, sin incorporarla todavía al score, de estimaciones EPS
forward.

Identificador futuro provisional:

eps_revision_3m_forward

Capturar, cuando la fuente lo permita:

- ticker;
- observed_at;
- fetched_at;
- forecast fiscal period;
- EPS estimate;
- analyst count;
- currency;
- source;
- content hash.

No calcular una revisión de tres meses hasta disponer de dos observaciones
comparables separadas por la ventana requerida.

No reconstruir historia anterior al primer snapshot.

Estado:

EPS_REVISION_TRUE_FACTOR =
FORWARD_HISTORY_REQUIRED

No sustituir target_rev_3m durante C2.0 inicial.

==================================================
3. MARGINS
==================================================

Aprobar como variante canónica inicial:

MARGINS_CANONICAL =
OPERATING_MARGIN_TTM_DELTA_YOY_PP

RAW_FORMULA =

operating_margin_ttm_current
-
operating_margin_ttm_comparable_prior_year

UNIT =
PERCENTAGE_POINTS

DIRECTION =
HIGHER_IS_BETTER

Los márgenes negativos y los deltas negativos son valores definidos y
adversos. No neutralizarlos.

Mantener como diagnósticos read-only:

MARGINS_DIAGNOSTIC_A =
OPERATING_MARGIN_TTM_LEVEL

MARGINS_DIAGNOSTIC_C =
50_PERCENT_LEVEL_PERCENTILE
+
50_PERCENT_DELTA_YOY_PERCENTILE

No usar A o C en el compuesto canónico sin una aprobación posterior.

Confirmar:

- definición TTM;
- periodos incluidos;
- comparabilidad YoY;
- unidad decimal o porcentual;
- acceptedDate de los estados utilizados.

==================================================
4. FCF
==================================================

Definición canónica:

FCF_YIELD =
FREE_CASH_FLOW_TTM / MARKET_CAP_AS_OF_SESSION

DIRECTION =
HIGHER_IS_BETTER

FCF negativo:

FACTOR_STATE =
KNOWN_ADVERSE_VALUE

Conservar el valor negativo.

No transformarlo en missing ni en neutral.

Si market cap es:

- ausente;
- cero;
- negativo;
- no certificable para la sesión;

clasificar:

FACTOR_STATE =
UNDEFINED_RATIO

PERCENTILE =
0.5

y registrar la razón.

Antes de marcar READY, entregar la fuente única y los campos exactos utilizados
para:

- operating cash flow;
- capital expenditure;
- FCF TTM;
- market cap;
- currency;
- price as-of.

No combinar silenciosamente proveedores o periodos incompatibles.

==================================================
5. POLÍTICAS DE NO FINITOS
==================================================

Clasificar cada observación como:

DEFINED_VALUE

TRUE_MISSING

UNDEFINED_RATIO

KNOWN_ADVERSE_VALUE

Nunca neutralizar KNOWN_ADVERSE_VALUE.

PE_HIST / PE_SECT

Si EPS <= 0 o PE <= 0:

FACTOR_STATE =
UNDEFINED_RATIO

PERCENTILE =
0.5

FLAG =
NON_POSITIVE_EARNINGS

No interpretar automáticamente como empresa cara.

Medir cuántos tickers reciben neutralidad por esta regla y reportarlo por
sector y capitalización.

Si la mediana de referencia es <= 0 o no certificable:

FACTOR_STATE =
UNDEFINED_REFERENCE

PERCENTILE =
0.5

DEBT

Si EBITDA > 0:

RAW_VALUE =
netDebt / EBITDA

DIRECTION =
LOWER_IS_BETTER

Si EBITDA <= 0 y netDebt > 0:

FACTOR_STATE =
KNOWN_ADVERSE_CAPITAL_STRUCTURE

Debe ordenarse peor que cualquier ratio válido, sin inventar un ratio
numérico infinito.

Si EBITDA <= 0 y netDebt <= 0:

FACTOR_STATE =
UNDEFINED_RATIO_WITH_NET_CASH

PERCENTILE =
0.5

FCF, MARGINS, TARGET REVISION Y SURPRISE

Los valores negativos definidos se conservan como señal adversa.

NaN e Infinity nunca se persisten como raw values válidos.

==================================================
6. BELOW_TGT
==================================================

Definición:

BELOW_TGT =

(consensus_target - session_price)
/
session_price

DIRECTION =
HIGHER_IS_BETTER

Si el target es ausente o <= 0:

FACTOR_STATE =
TRUE_MISSING_OR_UNDEFINED

PERCENTILE =
0.5

Si el precio es ausente o <= 0:

ELIGIBILITY_OR_DATA_QUALITY_STATUS =
UNCERTIFIABLE_PRICE

No calcular el factor.

Registrar por separado:

- target;
- session price;
- observed_at del target;
- price_asof;
- analyst count, si existe;
- source.

No asumir que target_rev_3m y below_tgt son independientes; reportar su
correlación durante la sombra.

==================================================
7. MA200
==================================================

Definición:

MA200_DISTANCE =

(adjusted_session_price - adjusted_ma200)
/
adjusted_ma200

DIRECTION =
HIGHER_IS_BETTER

Utilizar precios ajustados coherentemente.

Requerir 200 sesiones válidas.

Si existen menos de 200 sesiones o falta la serie:

FACTOR_STATE =
TRUE_MISSING

PERCENTILE =
0.5

FLAG =
INSUFFICIENT_PRICE_HISTORY_OR_DATA_QUALITY

Registrar si la ausencia contradice el requisito de diez años del roster.

No sustituir silenciosamente precios no ajustados por ajustados.

==================================================
8. SURPRISE
==================================================

Variante canónica inicial:

SURPRISE_MINIMUM_OBSERVATIONS =
4

RAW_VALUE =
media de las sorpresas porcentuales EPS de las últimas cuatro publicaciones
comparables disponibles antes del cutoff

Los valores negativos se conservan.

Si existen menos de cuatro observaciones comparables:

FACTOR_STATE =
PARTIAL_HISTORY

PERCENTILE =
0.5

Variante diagnóstica:

SURPRISE_DIAGNOSTIC_MINIMUM_OBSERVATIONS =
2

Comparar cobertura y estabilidad, pero no utilizarla en el compuesto canónico
sin autorización posterior.

Definir la política cuando la estimación EPS sea cero antes de marcar el
factor READY.

==================================================
9. PE_HIST Y PE_SECT
==================================================

Mantener las fórmulas propuestas:

PE_HIST =

(historical_5y_median_pe - current_pe)
/
historical_5y_median_pe

PE_SECT =

sector_median_pe - current_pe
/
sector_median_pe

Corregir cualquier ambigüedad de paréntesis en código y documentación:

PE_SECT =

(sector_median_pe - current_pe)
/
sector_median_pe

DIRECTION =
HIGHER_IS_BETTER

Antes de marcar READY, definir:

- fuente de current PE;
- fuente y construcción de la mediana histórica;
- ventana temporal real;
- mínimo de observaciones;
- tratamiento de corporate actions;
- universo utilizado para mediana sectorial;
- mínimo de empresas válidas por sector;
- cutoff y procedencia PIT.

No reconstruir medianas históricas con datos posteriores al cutoff.

==================================================
10. PRIMER CICLO PIT CERTIFICADO
==================================================

Autorizo ejecutar ahora el primer ciclo PIT certificado sobre:

C2_0_UNIVERSE =
CURATED_123

Fecha de referencia:

2026-07-28

Ejecutarlo exclusivamente mediante:

independent staged fetch
→ validation
→ completeness gate
→ seal

No utilizar:

backtest/.cache/fund_*.json

como fuente certificada.

No modificar esa caché.

El fetch debe preservar campos fuente suficientes para recalcular fórmulas
posteriormente.

Capturar source-native evidence, no solo valores derivados.

Para cada fuente guardar, dentro de los límites permitidos:

- campos crudos necesarios;
- timestamps;
- periodos;
- moneda;
- analyst count si existe;
- source identifier;
- extractor version;
- normalization version;
- content hash.

La decisión posterior de una fórmula no debe obligar a repetir o reinterpretar
silenciosamente el fetch de hoy.

==================================================
11. RESULTADO DEL CICLO PIT
==================================================

Si el ciclo completa todo el universo y supera validación:

PIT_CERTIFIED_COLLECTION =
COMPLETE

PIT_FORWARD_EFFECTIVE_FROM =
<cutoff exacto del snapshot>

Si existe rate limit, fallo de fuente o universo parcial:

PIT_CERTIFIED_COLLECTION =
INCOMPLETE_INPUT

PIT_FORWARD_EFFECTIVE_FROM =
NONE

Registrar:

- símbolos completos;
- símbolos fallidos;
- fuentes fallidas;
- campos ausentes;
- retries;
- timestamps;
- estado final.

No sellar parcialmente como COMPLETE.

Autorizar retries idempotentes únicamente para completar el mismo cutoff si
las fuentes siguen siendo temporalmente compatibles.

No mezclar datos obtenidos después con un cutoff anterior sin registrar la
diferencia de disponibilidad.

==================================================
12. COBERTURA
==================================================

La regla canónica permanece:

MINIMUM_CONTINUOUS_FACTORS_PRESENT =
9

Pero medir antes de interpretar C2.1:

- cobertura por factor;
- cobertura por sector;
- cobertura por capitalización;
- cobertura de factores de analistas;
- cobertura de factores fundamentales;
- cobertura de factores de mercado.

Bandas de capitalización:

300M_TO_1B
1B_TO_2B
2B_TO_10B
ABOVE_10B

Entregar pass rate de 9/11 para cada banda.

Ejecutar además el diagnóstico read-only:

NON_ANALYST_FACTOR_DIAGNOSTIC =
minimum 8 of 9 non-analyst factors

Los factores de analistas ausentes reciben 0.5 en esta variante diagnóstica.

No cambiar todavía la regla canónica 9/11.

==================================================
13. REV_GROW Y SECTOR
==================================================

Mantener como canónico:

REV_GROWTH_MIN =
0.08

Ejecutar diagnósticos read-only:

0.00
0.04
0.08

Reportar distribución sectorial antes y después de cada gate.

No presentar las variantes 0% o 4% como candidatas canónicas.

El objetivo es atribuir la concentración entre:

- composición del roster;
- tamaño;
- cobertura;
- gate de crecimiento;
- ranking.

==================================================
14. PRIORIDAD DE C2.0
==================================================

No retrasar C2.0 por el enriquecimiento completo de C2.1.

Orden:

1. primer PIT certificado;
2. cerrar fuentes y fórmulas de C2.0;
3. implementar C2.0 en sombra;
4. añadir histéresis, cadencia y sizing sombra;
5. validar revisiones y riesgo;
6. continuar C2.1.

C2.1 permanece:

PENDING_ELIGIBILITY_POLICY

==================================================
15. COMMITS
==================================================

Crear commits separados:

1.
docs(growth): approve initial continuous factor policies

2.
feat(growth): capture first certified forward PIT snapshot

3.
test(growth): verify first certified PIT snapshot

No implementar todavía el score continuo.

No hacer push.
No crear tag.

==================================================
16. CHECKPOINT
==================================================

Entregar:

TARGET_REV_3M =
APPROVED

TRUE_EPS_REVISION =
FORWARD_HISTORY_REQUIRED

MARGINS_CANONICAL =
OPERATING_MARGIN_TTM_DELTA_YOY_PP

SURPRISE_MINIMUM_OBSERVATIONS =
4

FORMULA_POLICIES_APPROVED =
PASS

FORMULA_SOURCE_CONTRACT =
PASS/PENDING_FIELDS

READY_FACTORS =
<lista>

BLOCKED_FACTORS =
<lista>

PIT_CERTIFIED_COLLECTION =
COMPLETE/INCOMPLETE_INPUT

PIT_FORWARD_EFFECTIVE_FROM =
<timestamp/NONE>

PIT_COMPLETE_SYMBOLS =
<número>

PIT_FAILED_SYMBOLS =
<número>

PIT_SOURCE_FAILURES =
<lista>

PIT_SOURCE_NATIVE_EVIDENCE =
PASS/FAIL

KNOWN_ADVERSE_VALUES_NEUTRALIZED =
NO

COVERAGE_9_OF_11_BY_MARKET_CAP =
<resumen>

ANALYST_COVERAGE_BY_MARKET_CAP =
<resumen>

REV_GROWTH_SECTOR_DIAGNOSTIC =
<resumen/PENDING_C2_1>

C2_0_STATUS =
PLANNED

C2_1_STATUS =
PENDING_ELIGIBILITY_POLICY

COMPLIANT_ENGINE_STATUS =
READY_NOT_ACTIVE

C0_C1_UNCHANGED =
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

No implementar el score continuo.
No activar ningún motor.
No desplegar.