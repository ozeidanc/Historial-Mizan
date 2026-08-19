CHECKPOINT FASE 1 APROBADO

COMMITS APROBADOS EN RAMA:

bc448af
5d5e224
68d3e40
13e63f9
ea6f90e

PRODUCTION_HEAD =
1d67c54

PRODUCTION_SCHEMA =
v31

C0_C1_UNCHANGED =
PASS

COMPLIANT_ENGINE_STATUS =
READY_NOT_ACTIVE

No activar el motor conforme.
No implementar todavía el score continuo.
No desplegar.

==================================================
1. ESTADO REAL DEL PIT
==================================================

Distinguir:

PIT_ARCHIVE_INFRASTRUCTURE =
READY

PIT_CERTIFIED_COLLECTION =
NOT_STARTED

La existencia del schema, estados y tests no significa todavía que se esté
acumulando historia PIT forward.

El historial PIT comienza únicamente con el primer snapshot:

- obtenido de fuentes staged;
- completo;
- validado;
- temporalmente coherente;
- sellado;
- con estado COMPLETE.

Entregar posteriormente:

PIT_FORWARD_EFFECTIVE_FROM =
<fecha y cutoff del primer snapshot completo>

==================================================
2. AUTORIZACIÓN SIGUIENTE
==================================================

Autorizo construir y ejecutar exclusivamente en LAB/staging:

A. primer ciclo de captura PIT forward;

B. auditoría exacta de campos y fórmulas de los 11 factores;

C. enriquecimiento read-only/staged del universo C2.1;

D. análisis de sensibilidad de elegibilidad.

No autorizar:

- C2 productivo;
- recomendaciones C2;
- refresh de fund_*.json productivo;
- activación de sealed-tiebreak-compliant;
- cambios en C0/C1;
- operaciones Wio.

==================================================
3. PROCEDENCIA TEMPORAL POR CLASE DE DATO
==================================================

No exigir acceptedDate a todas las fuentes.

Clasificar cada input como:

SEC_OR_FILING_FUNDAMENTAL

Temporalidad:

acceptedDate
period_end
fetched_at

MARKET_PRICE_DERIVED

Temporalidad:

price_asof
session_cutoff
fetched_at

ANALYST_ESTIMATE_OR_TARGET

Temporalidad:

observed_at
provider_asof
fetched_at
forecast_period

STATIC_REFERENCE

Temporalidad:

effective_from
fetched_at

No inventar acceptedDate para:

- precio objetivo;
- consenso de analistas;
- revisiones EPS;
- precios;
- MA200.

El PIT debe registrar el timestamp que prueba cuándo el valor estaba
disponible para Mizan.

==================================================
4. MISSING FRENTE A SEÑAL ADVERSA
==================================================

Distinguir:

TRUE_MISSING

El dato no existe o la fuente no lo entregó.

Tratamiento, si la cobertura total es >= 9/11:

percentile =
0.5

KNOWN_ADVERSE_VALUE

El dato existe y es económicamente desfavorable.

Ejemplos:

- FCF negativo;
- margen negativo;
- revisión EPS negativa;
- sorpresa negativa;
- crecimiento negativo;
- deuda elevada.

Tratamiento:

no imputar 0.5;
conservar como señal económica;
normalizar según la dirección del factor.

UNDEFINED_RATIO_WITH_ECONOMIC_INFORMATION

Ejemplos:

- P/E no significativo por EPS <= 0;
- netDebt/EBITDA no significativo por EBITDA <= 0;
- división por cero.

No clasificarlos automáticamente como missing neutral.

Para cada caso, proponer una política determinista que preserve el significado
económico, por ejemplo:

- peor percentil;
- métrica alternativa firmada;
- categoría adversa previa al percentile rank.

No aprobar ni implementar la política hasta entregar el preview.

==================================================
5. CONTRATO EXACTO POR FACTOR
==================================================

Para cada uno de los 11 factores entregar:

FACTOR_ID =
<id>

ECONOMIC_ROLE =
<descripción>

SOURCE_ENDPOINT_OR_ARTIFACT =
<fuente exacta>

SOURCE_FIELDS =
<campos exactos>

RAW_FORMULA =
<fórmula exacta>

UNIT =
<unidad>

DIRECTION =
HIGHER_IS_BETTER / LOWER_IS_BETTER

TEMPORAL_CLASS =
SEC_OR_FILING_FUNDAMENTAL /
MARKET_PRICE_DERIVED /
ANALYST_ESTIMATE_OR_TARGET /
STATIC_REFERENCE

AVAILABILITY_TIMESTAMP =
<campo>

PERIOD_FIELD =
<campo>

PIT_SAFE_NOW =
YES/NO/PARTIAL

FORWARD_HISTORY_REQUIRED =
YES/NO

TRUE_MISSING_POLICY =
<regla>

KNOWN_ADVERSE_POLICY =
<regla>

ZERO_DENOMINATOR_POLICY =
<regla>

COVERAGE_C2_0 =
<porcentaje>

COVERAGE_C2_1 =
<porcentaje>

FORMULA_STATUS =
READY /
REQUIRES_APPROVAL /
NOT_IMPLEMENTABLE_WITH_CURRENT_SOURCE

No inventar campos ni semántica.

==================================================
6. DECISIONES ESPECÍFICAS DE FÓRMULA
==================================================

MARGINS

Entregar tres previews separados:

A.
operating_margin_ttm

B.
operating_margin_delta_yoy

C.
combinación explícita 50/50 de sus percentile ranks

Para cada variante mostrar:

- cobertura;
- distribución;
- correlación;
- sectores favorecidos;
- impacto en ranking.

No elegir todavía.

EPS_REV

Definir exactamente:

- métrica revisada;
- horizonte de forecast;
- periodo fiscal;
- valor actual;
- valor comparable anterior;
- ventana de tres meses;
- número de analistas si existe;
- tratamiento de cambio de periodo fiscal.

No llamar eps_rev a una variación del precio objetivo.

Si no existe una observación histórica comparable de hace tres meses:

PIT_SAFE_NOW =
NO

FORWARD_HISTORY_REQUIRED =
YES

No reconstruirla usando el snapshot actual.

PE_HIST Y PE_SECT

Definir explícitamente:

- política para EPS <= 0;
- P/E <= 0;
- mediana <= 0;
- sectores con pocas observaciones;
- ausencia de cinco años completos;
- procedencia PIT de la mediana histórica.

DEBT

Definir:

- net debt;
- EBITDA;
- periodo;
- EBITDA <= 0;
- net cash;
- moneda y normalización.

FCF

Conservar FCF negativo como señal adversa.

No clasificarlo como missing.

BELOW_TGT

Definir:

- consensus target;
- observed_at;
- número mínimo de analistas;
- target <= 0;
- target ausente;
- caducidad del consenso.

MA200

Definir:

- precios ajustados;
- número mínimo de sesiones;
- corporate actions;
- missing price;
- menos de 200 observaciones.

SURPRISE

Definir:

- actual EPS;
- estimated EPS;
- últimas cuatro publicaciones;
- menos de cuatro observaciones;
- estimación cero;
- periodos comparables.

==================================================
7. PRIMER CICLO PIT EN LAB
==================================================

Ejecutar el primer ciclo para:

C2_0_UNIVERSE =
CURATED_123

No utilizar la caché productiva fund_*.json como fuente certificada.

Permitir fuentes staged nuevas.

El ciclo debe demostrar:

- cutoff único;
- universo completo;
- sin rate-limit parcial;
- acceptedDate <= cutoff donde aplique;
- timestamps de observación para estimaciones;
- precios con as-of de sesión;
- contenido inmutable;
- hash;
- lineage;
- estado COMPLETE o explicación de INCOMPLETE_INPUT.

No consumir el snapshot para seleccionar acciones todavía.

==================================================
8. ENRIQUECIMIENTO C2.1
==================================================

Autorizar enriquecimiento read-only/staged del roster completo.

No modificar roster-cache.json productivo.

Aplicar provisionalmente los escenarios:

MARKET_CAP =
300M / 1B / 2B

MEDIAN_20D_DOLLAR_VOLUME =
1M / 5M / 10M

PRICE_MINIMUM =
5 USD

SECTOR_GATE =
NONE

ADR =
EXCLUDE

TRADING_CURRENCY =
USD

COMMON_STOCK_ONLY =
YES

Estos son escenarios de sensibilidad, no parámetros finales.

Entregar por escenario:

- count inicial;
- enrichment completo;
- fallos;
- exclusiones;
- elegibles;
- cobertura 9/11;
- distribución sectorial por etapa;
- motivos de exclusión;
- frontera 25/26 potencial;
- nombres nuevos fuera de las 123.

No calcular retorno histórico como criterio de elección.

==================================================
9. EMBUDO SECTORIAL
==================================================

Para cada escenario reportar sector por sector después de:

1. roster bruto;
2. instrumento;
3. ADR/moneda;
4. capitalización;
5. liquidez;
6. precio;
7. fundamentales;
8. cobertura 9/11;
9. gates económicos;
10. elegibles.

Todavía no existe top 25 C2 hasta aprobar las fórmulas.

El objetivo es identificar si una concentración futura procede de:

- composición del roster;
- calidad de datos;
- gate de crecimiento;
- requisitos de tamaño/liquidez;
- factores del ranking.

==================================================
10. COMMITS
==================================================

Crear commits separados:

1.
docs(growth): document continuous factor source contracts

2.
feat(growth): stage first forward PIT input collection

3.
test(growth): verify certified PIT collection completeness

4.
feat(growth): add full-roster enrichment preview

No implementar el score continuo.

No hacer push.
No crear tag.

==================================================
11. CHECKPOINT
==================================================

Entregar:

FACTOR_FORMULA_CONTRACT =
PASS/PENDING_APPROVAL

READY_FACTORS =
<lista>

BLOCKED_FACTORS =
<lista>

PIT_ARCHIVE_INFRASTRUCTURE =
READY

PIT_CERTIFIED_COLLECTION =
COMPLETE/INCOMPLETE_INPUT/NOT_STARTED

PIT_FORWARD_EFFECTIVE_FROM =
<timestamp/NONE>

CURATED_123_PIT_COVERAGE =
<resumen>

ANALYST_FACTORS_WITH_FORWARD_HISTORY_REQUIRED =
<lista>

KNOWN_ADVERSE_VALUES_NEUTRALIZED =
NO

MARGINS_VARIANT_PREVIEW =
PASS/FAIL

EPS_REV_CONTRACT =
PASS/FAIL/PENDING_HISTORY

FULL_ROSTER_ENRICHMENT =
PASS/FAIL/INCOMPLETE_INPUT

MARKET_CAP_SENSITIVITY =
<resumen>

LIQUIDITY_SENSITIVITY =
<resumen>

SECTOR_ATTRITION_FUNNEL =
PASS/FAIL

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

EXACT_DECISIONS_REQUIRED_FROM_OMAR =
<lista/NONE>

Detenerse después del checkpoint.

No implementar el score continuo.
No activar ningún motor.
No desplegar.