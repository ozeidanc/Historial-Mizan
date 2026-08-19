MIZAN · RESOLVER BLOQUEOS ANTES DE IMPLEMENTAR C2

El contenido íntegro de:

spec-mizan-c2-seleccion.md

precede a esta orden y es normativo.

No implementar todavía el score continuo.
No refrescar la caché productiva.
No cambiar la selección activa.
No modificar C0/C1.
No generar recomendaciones.

==================================================
1. RECONCILIAR PANW Y LA CADENCIA
==================================================

Resolver la contradicción entre:

FASE 0:

NIGHTLY_FREEZE_CHANGES_ACTIVE_COMPOSITION = YES
PANW_EXIT_CAUSED_BY_NIGHTLY_REFREEZE = PASS

y el checkpoint posterior:

CADENCE_DEFECT = NOT_CONFIRMED

Trazar con IDs reales:

C0_SELECTION_FROZEN
→ selección cargada por generateDailyRecommendationSession
→ review session
→ recomendación PANW
→ decisión
→ recommendation_execution_link
→ movement_id
→ pérdida realizada.

Entregar:

PANW_FREEZE_ID =
<id>

PANW_SELECTION_ID_CONSUMED =
<id>

PANW_REVIEW_SESSION_ID =
<id>

PANW_RECOMMENDATION_ID =
<id>

PANW_ACTION =
<acción>

PANW_MOVEMENT_ID =
<id>

PANW_REALIZED_RESULT =
<importe>

GENERATOR_SELECTION_SOURCE =
DAILY_FREEZE /
QUARTERLY_ACTIVE_SELECTION /
OTHER

Si el generador consumió el freeze diario:

CADENCE_CLASSIFICATION =
DAILY_FREEZE_CHANGED_OPERATIONAL_COMPOSITION

Si no lo consumió:

CADENCE_CLASSIFICATION =
DAILY_FREEZE_ARCHIVAL_ONLY

y retirar formalmente la conclusión errónea anterior.

No basar la conclusión en el nombre “freeze”.

==================================================
2. CONFIRMAR LA POLÍTICA DE SOBRESCRITURA
==================================================

Mostrar el código exacto y el flujo de:

refresh-fundamentals.mjs

y de la escritura de:

backtest/.cache/fund_<ticker>.json

Responder:

EXISTING_POPULATED_CACHE_CAN_BE_REPLACED =
YES/NO

OVERWRITE_CONDITION =
<condición lógica exacta>

EMPTY_CACHE_REQUIRED_FOR_OVERWRITE =
YES/NO

FRESHER_ACCEPTED_DATE_CAUSES_OVERWRITE =
YES/NO

NEWER_SOURCE_PERIOD_CAUSES_OVERWRITE =
YES/NO

BETTER_FIELD_COVERAGE_CAUSES_OVERWRITE =
YES/NO

EQUAL_OR_WORSE_FETCH_BEHAVIOR =
<regla>

Crear únicamente tests aislados si hacen falta para demostrar la condición.

No ejecutar el refresh productivo.

Clasificar:

CACHE_REFRESH_POLICY =
WRITE_ONCE /
REPLACE_IF_BETTER /
REPLACE_IF_NEWER /
OTHER

==================================================
3. MEDIR HETEROGENEIDAD EN EL UNIVERSO ACTIVO
==================================================

No utilizar 860/1862 como métrica principal.

Medir para:

- las 123 acciones curadas;
- las acciones post-sector;
- las elegibles;
- las 25 seleccionadas.

Por ticker y factor distinguir:

- cache mtime;
- fetchedAt;
- acceptedDate;
- source period;
- price as-of;
- fecha económica del dato.

Entregar:

ACTIVE_123_DATE_DISTRIBUTION =
<resumen>

POST_SECTOR_DATE_DISTRIBUTION =
<resumen>

ELIGIBLE_DATE_DISTRIBUTION =
<resumen>

SELECTED_25_DATE_DISTRIBUTION =
<resumen>

CROSS_SECTIONAL_INPUT_HETEROGENEITY =
PASS/FAIL

TICKERS_USING_PRE_Q2_DATA =
<lista>

TICKERS_USING_POST_Q2_DATA =
<lista>

FACTORS_WITH_MIXED_ASOF_DATES =
<lista>

No llamar stale a un dato únicamente por su mtime.

Aplicar una política de frescura específica por tipo de factor.

==================================================
4. DESEMPATE SELLADO NO IMPLEMENTADO
==================================================

Verificar exactamente si el sello exige:

score_desc
→ greens_desc
→ revGrowthPct_desc
→ ticker_asc

y si el runtime ejecuta:

score_desc
→ greens_desc
→ ticker_asc

Entregar:

SEALED_TIEBREAK_KEYS =
<lista>

IMPLEMENTED_TIEBREAK_KEYS =
<lista>

REVGROWTHPCT_AVAILABLE_BEFORE_SORT =
YES/NO

REVGROWTHPCT_ACTUALLY_USED =
YES/NO

COMPLIANCE_DEFECT =
CONFIRMED/NOT_CONFIRMED

Reproducir para cada una de las ocho sesiones históricas:

- selección actual;
- selección usando el desempate sellado;
- altas;
- bajas;
- plazas antes alfabéticas;
- revGrowthPct de los candidatos empatados.

Auditar específicamente el grupo:

ABNB
AMD
CDNS
PANW
TTWO
WDAY

No modificar producción.

==================================================
5. PRESERVACIÓN HISTÓRICA DEL POSIBLE ARREGLO
==================================================

Si COMPLIANCE_DEFECT = CONFIRMED, determinar cómo corregirlo sin romper:

- C0 histórico;
- methodology_hash;
- hashes de selección;
- test dorado;
- revisiones históricas.

Evaluar:

A. VERSIONED_ENGINE_IMPLEMENTATION

La metodología mantiene el mismo hash contractual, pero las sesiones guardan:

selection_engine_version =
legacy / sealed-tiebreak-compliant

y la implementación nueva tiene effective_from futuro.

B. NEW_METHODOLOGY_ONLY

La infraestructura no permite reproducir de forma segura ambas
implementaciones; la corrección queda en C2.

No reemplazar silenciosamente la función compartida.

Entregar:

HISTORICAL_REPLAY_WITH_LEGACY_ENGINE_POSSIBLE =
YES/NO

FORWARD_COMPLIANT_ENGINE_POSSIBLE =
YES/NO

C0_GOLDEN_REMAINS_BYTE_IDENTICAL =
PASS/FAIL

RECOMMENDED_TIEBREAK_FIX_ROUTE =
A_VERSIONED_ENGINE /
B_C2_ONLY

==================================================
6. SNAPSHOT ANTES DEL REFRESH
==================================================

Diseñar el primer snapshot como:

AS_OBSERVED_INPUT_SNAPSHOT

Debe sellar exactamente qué datos consume hoy C0/C1:

- ticker;
- factor;
- raw value;
- cache mtime;
- fetchedAt;
- acceptedDate;
- period end;
- binary check;
- missing state;
- source;
- hashes.

No llamarlo histórico PIT retroactivo.

Declarar:

FORWARD_PIT_HISTORY_EFFECTIVE_FROM =
<fecha del primer snapshot regular>

No refrescar antes de capturar esta evidencia.

==================================================
7. DECISIONES YA RATIFICADAS
==================================================

C2_IS_NEW_METHODOLOGY =
YES

C0_C1_METHODOLOGY_FILES_MODIFIED =
NO

C0_C1_HASHES_MODIFIED =
NO

C2_VALIDATION =
FORWARD_SHADOW

PIT_ARCHIVE =
START_AS_SOON_AS_SAFELY_IMPLEMENTED

PRODUCTIVE_CACHE_REFRESH =
NOT_AUTHORIZED

C2_ACTIVATION =
NOT_AUTHORIZED

==================================================
8. ENTREGA
==================================================

NORMATIVE_SPEC_PRESENT =
PASS/FAIL

NORMATIVE_SPEC_REVIEWED =
PASS/FAIL

PANW_LINEAGE_COMPLETE =
PASS/FAIL

CADENCE_CLASSIFICATION =
DAILY_FREEZE_CHANGED_OPERATIONAL_COMPOSITION /
DAILY_FREEZE_ARCHIVAL_ONLY /
INSUFFICIENT_EVIDENCE

CACHE_REFRESH_POLICY =
WRITE_ONCE /
REPLACE_IF_BETTER /
REPLACE_IF_NEWER /
OTHER

EXISTING_POPULATED_CACHE_CAN_BE_REPLACED =
YES/NO

ACTIVE_UNIVERSE_INPUT_HETEROGENEITY =
PASS/FAIL

SEALED_TIEBREAK_IMPLEMENTED =
YES/NO

COMPLIANCE_DEFECT =
CONFIRMED/NOT_CONFIRMED

ALPHABETIC_SEATS_AFTER_SEALED_TIEBREAK =
<número>

PANW_RESULT_WITH_SEALED_TIEBREAK =
SELECTED/NOT_SELECTED

HISTORICAL_REPLAY_WITH_LEGACY_ENGINE_POSSIBLE =
YES/NO

RECOMMENDED_TIEBREAK_FIX_ROUTE =
A_VERSIONED_ENGINE /
B_C2_ONLY

AS_OBSERVED_SNAPSHOT_DESIGN =
PASS/FAIL

FILES_MODIFIED =
<documentación/tests únicamente o 0>

PRODUCTION_WRITES =
0

ACTIVE_SELECTION_CHANGED =
NO

Detenerse después de responder estas preguntas.

No comenzar Fase 1.