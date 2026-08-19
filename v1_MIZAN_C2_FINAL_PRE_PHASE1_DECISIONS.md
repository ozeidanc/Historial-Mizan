MIZAN · DECISIONES FINALES PREVIAS A FASE 1 C2

El archivo normativo:

spec-mizan-c2-seleccion.md

precede íntegramente a esta orden.

No comenzar todavía la implementación del score continuo.

==================================================
1. ESPECIFICACIÓN NORMATIVA
==================================================

Añadir el documento íntegro al repositorio mediante un commit exclusivamente
documental:

docs(growth): add normative C2 selection specification

Antes de continuar:

NORMATIVE_SPEC_PRESENT =
PASS

NORMATIVE_SPEC_REVIEWED =
PASS

PHASE_0_CONSISTENT_WITH_NORMATIVE_SPEC =
PASS/FAIL

Si la Fase 0 contradice el documento, entregar las diferencias y detenerse.

==================================================
2. DECISIÓN METODOLÓGICA
==================================================

Ratifico:

C2_IS_NEW_METHODOLOGY =
YES

C2 debe tener:

- archivo sellado nuevo;
- methodology_hash nuevo;
- estado SHADOW;
- cero recomendaciones ejecutables;
- cero impacto sobre C0/C1.

No modificar:

- C0;
- C1;
- sus archivos sellados;
- sus hashes;
- sus revisiones históricas;
- sus recomendaciones;
- sus selecciones activas.

La computación diaria C2 no implica cambio diario de composición.

C2 debe separar:

computed_selection =
diaria y observacional

active_shadow_selection =
trimestral, salvo regla dura futura expresamente autorizada

==================================================
3. DECISIÓN SOBRE FUNDAMENTALES
==================================================

No elegir entre:

A. refrescar;

o:

B. registrar staleness.

Autorizar ambas cosas de forma aislada y segura.

Implementar un proceso controlado que:

1. obtenga los fundamentales candidatos actuales;
2. no sobrescriba todavía la caché consumida por C0/C1;
3. materialice los resultados en staging o artefactos temporales;
4. compare datos actuales frente a datos candidatos;
5. selle el snapshot PIT inmutable;
6. lo use inicialmente solo para C2 sombra.

No cambiar la selección activa.

No generar recomendaciones productivas.

No ejecutar automáticamente:

refresh → freeze → review

como una sola operación.

Separar obligatoriamente:

DATA_FETCH

PIT_SNAPSHOT_SEAL

C2_SHADOW_COMPUTE

PRODUCTIVE_SELECTION_ACTIVATION

La última permanece prohibida.

==================================================
4. MEDICIÓN CORRECTA DE STALENESS
==================================================

No utilizar:

860 / 1862 archivos

como medida principal, porque incluye ficheros fuera del universo activo,
duplicados temporales o símbolos históricos potenciales.

Medir específicamente:

ACTIVE_CURATED_UNIVERSE =
123 aproximadamente

POST_SECTOR_UNIVERSE =
61 aproximadamente

ELIGIBLE_UNIVERSE =
36 aproximadamente

CURRENT_SELECTION =
25

Para cada conjunto entregar:

TOTAL_TICKERS =
<número>

CACHE_FILES_FOUND =
<número>

CACHE_FILES_MISSING =
<número>

CACHE_MTIME_OLDER_THAN_30_DAYS =
<número>

SOURCE_ACCEPTED_DATE_STALE =
<número>

LATEST_SOURCE_PERIOD_STALE =
<número>

FETCHED_AT_STALE =
<número>

FACTORS_AFFECTED =
<lista>

Distinguir por ticker y factor:

- cache_mtime;
- fetched_at;
- acceptedDate;
- source period end;
- precio as-of;
- fecha económica del dato;
- frecuencia esperada de actualización.

No afirmar que un filing está fechado el 21 de junio únicamente porque el
cache mtime sea del 21 de junio.

==================================================
5. POLÍTICA DE FRESCURA POR FACTOR
==================================================

Clasificar cada factor:

QUARTERLY_FUNDAMENTAL

ANALYST_ESTIMATE

MARKET_PRICE_DERIVED

CORPORATE_EVENT

STATIC_REFERENCE

Para cada factor indicar:

EXPECTED_UPDATE_CADENCE =
<frecuencia>

MAX_ACCEPTABLE_DATA_AGE =
<duración o NONE>

CURRENT_SOURCE_DATE =
<fecha>

CURRENT_FETCH_DATE =
<fecha>

STALE =
YES/NO/UNKNOWN

IMPACT_IF_STALE =
<descripción>

Prestar especial atención a:

- eps_rev;
- target price / below_tgt;
- surprise;
- ma200;
- revGrowthPct;
- margins;
- FCF;
- debt;
- P/E histórico y sectorial.

No aplicar un TTL único arbitrario a todos los factores.

==================================================
6. REFRESH CONTROLADO
==================================================

Ejecutar primero en LAB o staging un refresh completo del universo activo.

No escribir en la caché productiva.

Entregar comparación before/after:

- ticker;
- factor;
- valor actual;
- valor candidato;
- source date actual;
- source date candidata;
- check actual;
- check candidato;
- score binario actual;
- score candidato;
- elegibilidad actual;
- elegibilidad candidata;
- rank actual;
- rank candidato;
- selección actual;
- selección candidata.

Resumir:

TICKERS_WITH_CHANGED_RAW_INPUTS =
<número>

TICKERS_WITH_CHANGED_CHECKS =
<número>

TICKERS_WITH_CHANGED_ELIGIBILITY =
<número>

TICKERS_WITH_CHANGED_RANK =
<número>

TOP25_ADDITIONS_IF_REFRESHED =
<lista>

TOP25_REMOVALS_IF_REFRESHED =
<lista>

No aplicar esos cambios a Producción.

==================================================
7. SNAPSHOT PIT
==================================================

El primer snapshot C2 debe capturar los datos recién obtenidos y validados,
no limitarse a copiar silenciosamente una caché potencialmente vencida.

Persistir por ticker y factor:

- raw value;
- source;
- acceptedDate;
- period end;
- fetchedAt;
- price as-of;
- expected cadence;
- staleness status;
- missing reason;
- binary check;
- continuous input;
- content hash.

Si un factor está vencido:

- no ocultarlo;
- aplicar la política missing/stale aprobada;
- registrar la razón.

No utilizar automáticamente un valor vencido como si fuera fresco.

==================================================
8. CONTRADICCIÓN SOBRE PANW Y LA CADENCIA
==================================================

Resolver antes de concluir si existe un bug de cadencia.

La Fase 0 declaró:

PANW_EXIT_CAUSED_BY_NIGHTLY_REFREEZE =
PASS

NIGHTLY_FREEZE_CHANGES_ACTIVE_COMPOSITION =
YES

El checkpoint posterior declaró:

CADENCE_DEFECT =
NOT_CONFIRMED

y afirmó que los freezes diarios no ejecutaron operaciones fuera de cadencia.

Trazar la cadena exacta de PANW:

1. selección activa anterior;
2. freeze/audit ID de cada sesión;
3. ranking y selección del 24 y 27 de julio;
4. review/recommendation session;
5. recommendation_id de PANW;
6. acción recomendada;
7. razón persistida;
8. decisión;
9. ejecución Wio;
10. recommendation_execution_link;
11. movement_id;
12. fecha y pérdida realizada;
13. función que cargó la selección utilizada para generar ELIMINAR.

Entregar:

PANW_PREVIOUS_ACTIVE_SELECTION_ID =
<id>

PANW_2026_07_27_FREEZE_ID =
<id>

PANW_RECOMMENDATION_SESSION_ID =
<id>

PANW_ELIMINATE_RECOMMENDATION_ID =
<id>

PANW_MOVEMENT_ID =
<id>

SELECTION_READ_BY_RECOMMENDATION_GENERATOR =
<id/tipo>

SELECTION_ORIGIN =
DAILY_FREEZE /
QUARTERLY_ACTIVE_SELECTION /
OTHER

==================================================
9. CLASIFICACIÓN DE CADENCIA
==================================================

Clasificar según evidencia:

A. DAILY_FREEZE_WAS_ARCHIVAL_ONLY

El freeze no fue consumido para cambiar la composición operativa.

En ese caso:

- CADENCE_DEFECT = NOT_CONFIRMED;
- explicar la causa real de ELIMINAR PANW;
- retirar formalmente la conclusión anterior de Fase 0.

B. DAILY_FREEZE_CHANGED_RECOMMENDATION_COMPOSITION

El generador leyó el freeze diario y produjo una entrada/salida operativa.

En ese caso:

- CADENCE_DEFECT = CONFIRMED_RUNTIME_BUG;
- el runtime contradice la frecuencia trimestral sellada;
- proponer hotfix separado;
- no implementarlo todavía.

C. EVIDENCE_INSUFFICIENT

No puede probarse el vínculo.

En ese caso:

- no afirmar ni negar el bug;
- enumerar los artefactos faltantes.

El nombre “freeze” o “snapshot” no determina la clasificación.

La clasificación depende de si su resultado fue consumido por el generador de
recomendaciones y produjo un cambio de composición.

==================================================
10. RELACIÓN CON EL HASH
==================================================

Se acepta que:

HASH_INCLUDES_REVIEW_FREQUENCY =
YES

HASH_INCLUDES_FREEZE_CADENCE =
NO

HASH_INCLUDES_ACTIVE_SELECTION_TRANSITION_RULES =
NO

Esto no basta por sí solo para decidir el hotfix.

Si el freeze diario cambió composición fuera de la frecuencia trimestral, el
hotfix propuesto no debe cambiar la frecuencia sellada: debe hacer que el
runtime la cumpla.

Sin embargo, no aplicarlo todavía hasta conocer:

- impacto sobre la selección vigente;
- próxima fecha trimestral;
- recomendaciones abiertas;
- fills pendientes;
- compatibilidad con C1;
- test dorado.

Cualquier hotfix debe ir en rama separada:

hotfix/growth-quarterly-active-selection

No mezclarlo con C2.

==================================================
11. CHECKPOINT
==================================================

Entregar:

NORMATIVE_SPEC_PRESENT =
PASS/FAIL

NORMATIVE_SPEC_CONSISTENT =
PASS/FAIL

C2_IS_NEW_METHODOLOGY =
YES

C0_C1_UNCHANGED =
PASS/FAIL

ACTIVE_UNIVERSE_STALENESS =
<resumen>

SELECTED_25_STALENESS =
<resumen>

FRESHNESS_POLICY_DEFINED =
PASS/FAIL

CONTROLLED_REFRESH_PREVIEW =
PASS/FAIL

TOP25_CHANGED_BY_REFRESH_PREVIEW =
YES/NO

PIT_SNAPSHOT_DESIGN =
PASS/FAIL

PIT_ARCHIVE_READY_TO_IMPLEMENT =
YES/NO

PANW_LINEAGE_COMPLETE =
PASS/FAIL

PANW_EXIT_SELECTION_ORIGIN =
DAILY_FREEZE /
QUARTERLY_SELECTION /
OTHER /
UNKNOWN

CADENCE_CLASSIFICATION =
A/B/C

PREVIOUS_PHASE0_CADENCE_CLAIM =
CONFIRMED/RETRACTED/UNPROVEN

CADENCE_HOTFIX_RECOMMENDED =
YES/NO/UNDETERMINED

FILES_MODIFIED =
<solo documento normativo o 0>

PRODUCTION_WRITES =
0

ACTIVE_SELECTION_CHANGED =
NO

Detenerse después de esta entrega.

No implementar todavía el score continuo.
No sustituir la caché productiva.
No activar C2.
No generar recomendaciones.