MIZAN · ESPECIFICACIÓN CORREGIDA Y SIGUIENTE FASE DE CONSTRUCCIÓN

BASELINE

BRANCH =
feature/c2-continuous-scoring

BRANCH_HEAD =
bc448af

PRODUCTION_HEAD =
1d67c54

PRODUCTION_SCHEMA =
v31

El checkpoint del motor versionado queda aprobado.

LEGACY_ENGINE =
BYTE_IDENTICAL

SEALED_TIEBREAK_COMPLIANT =
READY_NOT_ACTIVE

C0_C1_CHANGED =
NO

PRODUCTION_DEPLOY =
NO

==================================================
1. CORRECCIÓN DEL DOCUMENTO NORMATIVO
==================================================

Crear en el repositorio:

spec-mizan-c2-seleccion.md

El documento debe conservar íntegramente:

- PARTE A · Diagnóstico;
- PARTE B · Restricciones de arquitectura;
- orden FASE 0 → FASE 1 → FASE 2 → FASE 3;
- prohibición de modificar C0/C1;
- modo sombra obligatorio;
- seleccionar() como función pura;
- append-only;
- test dorado;
- prohibición de backtest contaminado;
- pesos iguales;
- validación forward.

Pero debe corregir el contrato original de factores.

La versión anterior confundía:

11 factores

con:

15 checks binarios.

El contrato normativo correcto es el siguiente.

==================================================
2. CONTRATO NORMATIVO · 15 CHECKS Y 11 FACTORES
==================================================

ACTIVE_BINARY_CHECKS =

1. pe_hist
2. pe_sect
3. below_tgt
4. debt
5. margins
6. fcf
7. eps_rev
8. rev_grow
9. surprise
10. ma200
11. mcap
12. coverage
13. insider
14. piotroski
15. dividend

Todos los checks binarios se conservan para:

- compatibilidad C0/C1;
- elegibilidad;
- explicación;
- UI;
- auditoría;
- diagnóstico.

C0 y C1 permanecen intactos.

CONTINUOUS_RANKING_FACTORS =

1. pe_hist
2. pe_sect
3. below_tgt
4. debt
5. margins
6. fcf
7. eps_rev
8. rev_grow
9. surprise
10. ma200
11. piotroski

Estos once factores forman inicialmente el compuesto continuo de C2.

Todos tienen pesos iguales.

No optimizar pesos en esta iteración.

==================================================
3. CHECKS FUERA DEL COMPUESTO
==================================================

mcap:

C2_ROLE =
ELIGIBILITY_GATE

La capitalización se utiliza para definir el universo elegible y para los
análisis de sensibilidad.

No recibe peso adicional en el score continuo.

coverage:

C2_ROLE =
DATA_SUFFICIENCY_GATE

Mide la suficiencia de inputs.

No es una característica económica de la empresa y no entra en el compuesto.

insider:

C2_ROLE =
DISPLAY_AND_DIAGNOSTIC_ONLY

Motivo:

cobertura observada aproximada del 3 %.

No entra inicialmente en el compuesto.

dividend:

C2_ROLE =
DISPLAY_AND_DIAGNOSTIC_ONLY

Motivos:

- cobertura parcial aproximada del 56 %;
- semántica discutible en una metodología de crecimiento;
- posible introducción de un sesgo de estilo no definido.

No eliminar estos checks del sistema, UI o archivo PIT.

Solo quedan fuera del compuesto continuo inicial.

==================================================
4. CORRECCIÓN CONCEPTUAL SOBRE DISCRIMINACIÓN
==================================================

La tasa de verdes de un check binario no determina si su magnitud continua
sirve para ordenar.

Regla normativa:

BINARY_CHECK_SATURATION
DOES_NOT_IMPLY
CONTINUOUS_FACTOR_NON_DISCRIMINATION

No retirar automáticamente un factor porque:

green_rate > 90 %

o:

green_rate < 10 %

La evidencia de rev_grow demuestra la diferencia:

Binary:

todas las elegibles pasan el gate de crecimiento.

Continuous:

ABNB =
12.6 %

PANW =
19.5 %

La magnitud continua conserva señal económica y resolvió los empates que el
check binario no podía resolver.

Por tanto, permanecen inicialmente en el compuesto continuo:

- rev_grow;
- below_tgt;
- debt;
- fcf;
- surprise.

Evaluar su utilidad mediante:

- dispersión de valores crudos;
- cobertura;
- estabilidad;
- procedencia PIT;
- correlación;
- contribución al ranking sombra.

No retirarlos usando únicamente la tasa de verdes.

==================================================
5. DOBLE ROL DE REV_GROW
==================================================

rev_grow tiene dos roles expresamente autorizados.

ELIGIBILITY_ROLE:

revGrowthPct >= 0.08

RANKING_ROLE:

percentile rank transversal de revGrowthPct dentro del universo elegible.

DIRECTION:

HIGHER_IS_BETTER

El gate exige un mínimo de crecimiento.

El factor continuo ordena a las empresas que ya superaron ese mínimo.

No considerar esta doble utilización un error o una duplicación accidental.

Debe quedar documentada en el archivo sellado C2.

==================================================
6. COBERTURA MÍNIMA
==================================================

APPROVED_CONTINUOUS_FACTOR_COUNT =
11

MINIMUM_CONTINUOUS_FACTOR_COVERAGE =
80_PERCENT

Regla determinista:

MINIMUM_CONTINUOUS_FACTORS_PRESENT =
9

Si un ticker tiene al menos nueve factores continuos presentes:

- puede permanecer elegible;
- cada factor ausente recibe exactamente 0.5;
- se registra missing reason;
- el denominador del compuesto permanece fijo en 11.

Si tiene ocho o menos:

ELIGIBILITY_STATUS =
INSUFFICIENT_FACTOR_COVERAGE

No calcular:

sum(present factors) / number of present factors

El compuesto siempre tiene denominador fijo de 11.

coverage no se añade como factor número 12.

==================================================
7. COMPUESTO CONTINUO
==================================================

Para cada uno de los once factores:

1. obtener la magnitud económica cruda;
2. comprobar semántica y unidad;
3. comprobar fecha económica;
4. aplicar acceptedDate <= cutoff;
5. tratar valores no finitos;
6. winsorizar transversalmente;
7. calcular percentile rank;
8. corregir la dirección;
9. asignar 0.5 si el factor falta y se supera la cobertura mínima;
10. calcular la media aritmética simple de los once percentiles.

CONTINUOUS_COMPOSITE_SCORE =

sum(11 factor percentiles) / 11

No convertir:

g / a / r

en números artificiales para construir el continuo.

El score debe partir de los valores económicos originales.

==================================================
8. WINSORIZACIÓN Y PERCENTILES
==================================================

No implementar todavía el score hasta entregar el contrato exacto de fórmulas.

Proponer de forma determinista:

WINSORIZATION =
P1 / P99 transversal por sesión y universo elegible

PERCENTILE_RANGE =
[0,1]

MISSING_PERCENTILE =
0.5 exacto

Documentar y probar posteriormente:

- método de interpolación;
- empates;
- un solo valor;
- todos los valores iguales;
- pocos valores;
- NaN;
- Infinity;
- dirección HIGHER_IS_BETTER;
- dirección LOWER_IS_BETTER.

Ticker no puede alterar el percentile rank económico.

==================================================
9. CORRELACIÓN Y REDUNDANCIA
==================================================

No retirar ni reducir pesos por correlación en esta iteración.

Reportar especialmente:

- pe_hist frente a pe_sect;
- pe_hist frente a ma200;
- pe_sect frente a eps_rev;
- eps_rev frente a ma200;
- rev_grow frente a margins;
- rev_grow frente a fcf;
- value frente a momentum.

La posible redundancia se evalúa durante la sombra forward.

No optimizar utilizando:

- ocho sesiones históricas;
- datos no PIT;
- backtests contaminados;
- retorno in-sample.

==================================================
10. SALIDA DE AUDITORÍA
==================================================

Los snapshots y audit trails deben separar expresamente:

binary_checks

continuous_raw_values

continuous_percentiles

continuous_factor_coverage

continuous_composite_score

eligibility_gates

display_only_checks

risk_only_checks

missing_reasons

Esto impide volver a confundir los 15 checks con los 11 factores.

==================================================
11. COMMIT DOCUMENTAL
==================================================

Crear un commit exclusivamente documental:

docs(growth): add corrected normative C2 selection specification

No mezclar código en ese commit.

Después declarar:

NORMATIVE_SPEC_PRESENT =
PASS

NORMATIVE_SPEC_REVIEWED =
PASS

ACTIVE_BINARY_CHECK_COUNT =
15

CONTINUOUS_RANKING_FACTOR_COUNT =
11

ELIGIBILITY_OR_COVERAGE_ONLY_COUNT =
2

DISPLAY_AND_DIAGNOSTIC_ONLY_COUNT =
2

MINIMUM_CONTINUOUS_FACTORS_PRESENT =
9

==================================================
12. AS_OBSERVED_INPUT_SNAPSHOT
==================================================

Después del commit documental, construir:

AS_OBSERVED_INPUT_SNAPSHOT

Propósito:

capturar exactamente los datos que C0/C1 consume actualmente antes de
cualquier refresh.

No refrescar antes:

- fundamentales;
- roster;
- cachés;
- filings;
- inputs productivos.

Registrar por sesión, ticker y factor:

- session_date;
- cutoff;
- methodology;
- methodology_hash;
- selection_engine_version;
- universe variant;
- ticker;
- factor_id;
- raw_value;
- binary_check;
- missing_state;
- missing_reason;
- cache/source identifier;
- cache_mtime;
- fetchedAt;
- acceptedDate;
- period_end;
- price_asof;
- source;
- staleness classification;
- input hash;
- row content hash;
- snapshot content hash.

Registrar además:

- universo considerado;
- elegibles;
- excluidos;
- motivos de exclusión;
- ranking;
- selección calculada;
- selección activa consumida.

Semántica:

AS_OBSERVED_SEMANTICS =
REPRODUCIBLE_OBSERVATION_OF_ACTUAL_ENGINE_INPUTS

No presentarlo como histórico PIT retroactivo.

==================================================
13. PIT FORWARD ARCHIVE
==================================================

Construir un archivo PIT independiente y append-only.

No implementarlo como copia diaria de:

backtest/.cache/fund_*.json

porque esa caché tiene política WRITE_ONCE.

Separar:

AS_OBSERVED_SOURCE =
CURRENT_C0_C1_INPUTS

PIT_FORWARD_SOURCE =
SESSION_SPECIFIC_STAGED_FETCH_OR_CERTIFIED_SOURCE

Pipeline:

external fetch
→ independent staging
→ validation
→ completeness check
→ point-in-time seal
→ future shadow consumer

No sobrescribir la caché productiva.

No conectar todavía C2 al pipeline.

==================================================
14. ESTADOS E INTEGRIDAD PIT
==================================================

Estados mínimos:

IN_PROGRESS
COMPLETE
INCOMPLETE_INPUT
SOURCE_FAILURE
CONTENT_CONFLICT

Reglas:

- mismo identificador y mismo contenido → idempotente;
- mismo identificador y contenido diferente → CONTENT_CONFLICT;
- snapshot parcial → no consumible por C2 canónica;
- rate limit → no sellar como COMPLETE;
- cutoffs mezclados → no sellar;
- acceptedDate posterior al cutoff → excluir esa evidencia;
- retries → completar únicamente partes faltantes;
- nunca producir un universo parcial silencioso.

El diseño debe soportar:

C2_0_UNIVERSE =
CURATED_123

C2_1_UNIVERSE =
FULL_ROSTER

No hardcodear el tamaño del universo.

==================================================
15. FÓRMULAS CONTINUAS · CHECKPOINT OBLIGATORIO
==================================================

Antes de implementar el score continuo, entregar para cada uno de los once
factores:

FACTOR_ID =
<id>

ECONOMIC_MEANING =
<descripción>

RAW_VALUE =
<campo/fórmula>

RAW_SOURCE =
<fuente>

UNIT =
<unidad>

DIRECTION =
HIGHER_IS_BETTER / LOWER_IS_BETTER

PIT_STATUS =
SAFE / UNSAFE / PARTIAL / UNKNOWN

COVERAGE_C2_0 =
<porcentaje>

COVERAGE_C2_1 =
<porcentaje/pendiente>

NON_FINITE_POLICY =
<regla>

PROPOSED_CONTINUOUS_FORMULA =
<fórmula>

WINSORIZATION_BEHAVIOR =
<regla>

MISSING_POLICY =
0.5 / STRUCTURAL_EXCLUSION

No inventar fórmulas.

Si un valor crudo no está disponible o su semántica es ambigua:

FORMULA_STATUS =
REQUIRES_APPROVAL

Detenerse antes de implementar ese factor.

==================================================
16. C2.0 Y C2.1
==================================================

No construir todavía el score continuo de C2.0 o C2.1 antes de aprobar el
checkpoint de fórmulas.

Mantener:

C2_0_STATUS =
PLANNED

C2_1_STATUS =
INCOMPLETE_INPUT

El enriquecimiento completo de las 1.060 acciones sigue diferido hasta cerrar:

- fórmulas;
- cobertura;
- staging;
- política de instrumentos;
- política de capitalización;
- política de liquidez.

==================================================
17. MOTOR VERSIONADO
==================================================

Mantener el estado actual:

LEGACY_ENGINE =
ACTIVE_DEFAULT_IN_CODE

SEALED_TIEBREAK_COMPLIANT =
READY_NOT_ACTIVE

No activar el motor conforme en Producción.

No cambiar sus call-sites productivos.

No mezclar su posible activación con C2.

Conservar:

- replay legacy 8/8;
- golden hash;
- methodology_hash;
- test 14/0;
- effective_from no configurado.

==================================================
18. TESTS
==================================================

Probar:

1. AS_OBSERVED completo;
2. idempotencia;
3. conflicto de contenido;
4. ticker ausente;
5. factor ausente;
6. universe partial;
7. source failure;
8. rate limit;
9. cutoffs incompatibles;
10. acceptedDate posterior al cutoff;
11. snapshot WRITE_ONCE capturado correctamente;
12. PIT independiente de fund_*.json productivo;
13. denominador fijo de 11;
14. cobertura 9/11 elegible;
15. cobertura 8/11 no elegible;
16. missing individual = 0.5;
17. C0/C1 sin cambios;
18. legacy golden sin cambios;
19. compliant READY_NOT_ACTIVE;
20. cero recomendaciones;
21. cero movimientos económicos;
22. cero órdenes Wio.

==================================================
19. COMMITS
==================================================

Crear commits separados:

1.
docs(growth): add corrected normative C2 selection specification

2.
feat(growth): add as-observed selection input snapshots

3.
feat(growth): add append-only forward PIT archive

4.
test(growth): verify C2 input snapshot integrity

No implementar todavía el score continuo.

No hacer push.

No crear tag.

==================================================
20. CHECKPOINT FINAL
==================================================

Entregar:

NORMATIVE_SPEC_PRESENT =
PASS/FAIL

NORMATIVE_SPEC_CONSISTENT =
PASS/FAIL

ACTIVE_BINARY_CHECK_COUNT =
15

CONTINUOUS_RANKING_FACTOR_COUNT =
11

MINIMUM_CONTINUOUS_FACTORS_PRESENT =
9

FACTOR_FORMULA_CONTRACT =
PASS/FAIL/PENDING_APPROVAL

AS_OBSERVED_IMPLEMENTED =
PASS/FAIL

AS_OBSERVED_FIRST_SNAPSHOT =
LAB_ONLY/NOT_CAPTURED

PIT_FORWARD_ARCHIVE_IMPLEMENTED =
PASS/FAIL

PIT_USES_INDEPENDENT_STAGING =
PASS/FAIL

PIT_DEPENDS_ON_PRODUCTIVE_FUND_CACHE =
NO

PIT_COMPLETENESS_GATE =
PASS/FAIL

PIT_IDEMPOTENCY =
PASS/FAIL

PIT_CONTENT_CONFLICT =
PASS/FAIL

CURATED_123_SUPPORTED =
PASS/FAIL

FULL_ROSTER_SCHEMA_SUPPORTED =
PASS/FAIL

LEGACY_GOLDEN_HASH_UNCHANGED =
PASS/FAIL

CURRENT_GOLDEN_SESSIONS =
8

PROMOTION_TARGET_GOLDEN_SESSIONS =
10

COMPLIANT_ENGINE_STATUS =
READY_NOT_ACTIVE

C2_0_STATUS =
PLANNED

C2_1_STATUS =
INCOMPLETE_INPUT

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

Detenerse después de este checkpoint.

No implementar el score continuo.
No activar el motor conforme.
No desplegar.
No refrescar la caché productiva.