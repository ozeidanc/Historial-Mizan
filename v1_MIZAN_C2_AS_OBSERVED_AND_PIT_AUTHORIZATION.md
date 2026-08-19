CHECKPOINT DEL MOTOR VERSIONADO APROBADO

Commit revisado:

bc448af

Estado aprobado:

LEGACY_ENGINE =
BYTE_IDENTICAL

SEALED_TIEBREAK_COMPLIANT =
READY_NOT_ACTIVE

C0_C1_CHANGED =
NO

PRODUCTION_DEPLOY =
NO

No activar todavía el motor conforme en Producción.

==================================================
1. ESPECIFICACIÓN NORMATIVA
==================================================

El contenido íntegro de:

spec-mizan-c2-seleccion.md

precede a esta orden.

Añadirlo al repositorio mediante un commit exclusivamente documental:

docs(growth): add normative C2 selection specification

Después declarar:

NORMATIVE_SPEC_PRESENT =
PASS

NORMATIVE_SPEC_REVIEWED =
PASS

PHASE_0_AND_CURRENT_DESIGN_CONSISTENT_WITH_SPEC =
PASS/FAIL

Si existe alguna contradicción material:

- documentarla;
- detener la implementación del score continuo;
- no resolverla silenciosamente.

==================================================
2. SIGUIENTE PASO AUTORIZADO
==================================================

Autorizo construir ahora, en:

feature/c2-continuous-scoring

y sin despliegue:

A. AS_OBSERVED_INPUT_SNAPSHOT

B. PIT_FORWARD_ARCHIVE

Estas construcciones deben incluir:

- código;
- schema o persistencia aditiva;
- tests;
- scripts de verificación;
- artefactos de LAB;
- cero activación productiva.

No comenzar todavía C2.0 ni C2.1 hasta cerrar el contrato normativo de
factores.

==================================================
3. AS_OBSERVED_INPUT_SNAPSHOT
==================================================

El primer snapshot debe capturar exactamente los inputs consumidos actualmente
por C0/C1.

No refrescar antes:

- fundamentales;
- roster;
- precios históricos;
- cachés;
- filings.

Propósito:

AUDIT_WHAT_THE_ENGINE_ACTUALLY_OBSERVED

Registrar por sesión, ticker y factor:

- session_date;
- cutoff;
- portfolio/methodology;
- selection_engine_version;
- ticker;
- factor_id;
- raw_value;
- binary_check;
- missing_state;
- cache path o source identifier;
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

El snapshot debe conservar también:

- universo considerado;
- elegibles;
- excluidos;
- motivo de exclusión;
- ranking;
- selección calculada;
- selección activa consumida.

No presentarlo como histórico PIT retroactivo.

Declarar:

AS_OBSERVED_SEMANTICS =
REPRODUCIBLE_OBSERVATION_OF_CURRENT_ENGINE_INPUTS

==================================================
4. DIFERENCIA ENTRE AS_OBSERVED Y PIT
==================================================

No implementar PIT_FORWARD_ARCHIVE como una mera copia diaria de:

backtest/.cache/fund_*.json

porque esa caché es WRITE_ONCE.

Copiar cada día el mismo dato congelado permitiría reproducir lo observado,
pero no produciría fundamentales point-in-time actuales para validar C2.

Separar:

AS_OBSERVED_SOURCE =
CURRENT_C0_C1_INPUTS

PIT_FORWARD_SOURCE =
SESSION_SPECIFIC_STAGED_FETCH_OR_CERTIFIED_SOURCE

El archivo PIT debe poder registrar datos nuevos disponibles en cada cutoff
sin modificar la fuente productiva de C0/C1.

==================================================
5. PIPELINE PIT FORWARD
==================================================

Diseñar el pipeline:

external fetch
→ staging
→ validation
→ completeness check
→ point-in-time seal
→ shadow consumer

No conectar todavía este pipeline a Producción.

No sobrescribir:

backtest/.cache/fund_<ticker>.json

No cambiar:

- C0/C1;
- selección activa;
- freezes C0;
- recomendaciones;
- holdings;
- cash;
- NAV;
- movimientos.

El staging debe ser independiente y append-only.

Cada snapshot PIT debe incluir:

- fecha de consulta;
- cutoff;
- fecha económica;
- acceptedDate;
- period end;
- fuente;
- payload o evidencia normalizada;
- raw factor values;
- cobertura;
- missing reasons;
- content hash;
- lineage;
- versión del extractor;
- versión de normalización.

==================================================
6. COMPLETITUD E IDEMPOTENCIA
==================================================

Una sesión no debe sellarse como PIT completa si:

- se alcanza un rate limit;
- falla parte del universo;
- hay respuestas parciales;
- faltan símbolos requeridos;
- se mezclan cutoffs;
- no puede verificarse la fuente.

Estados mínimos:

IN_PROGRESS
COMPLETE
INCOMPLETE_INPUT
SOURCE_FAILURE
CONTENT_CONFLICT

Reglas:

- mismo session/ticker/factor y mismo contenido → idempotente;
- mismo identificador y contenido distinto → CONTENT_CONFLICT;
- snapshot incompleto → no consumible por C2 canónica;
- retry → completa únicamente las partes faltantes;
- ningún fallo produce un universo parcial silencioso.

==================================================
7. ALCANCE INICIAL Y DISEÑO ESCALABLE
==================================================

La primera verificación puede ejecutarse sobre el universo C2.0:

CURATED_123

Pero el modelo de datos y el pipeline deben soportar:

FULL_ROSTER =
1060 actualmente

No hardcodear 123 ni 1060.

No realizar todavía el enriquecimiento completo de las 1.060 acciones.

Demostrar mediante tests que:

- el schema no depende del tamaño del universo;
- registra exclusiones;
- detecta snapshots parciales;
- conserva lineage por variante.

==================================================
8. CONSUMO POR METODOLOGÍA
==================================================

Los snapshots deben distinguir:

C0_C1_AS_OBSERVED

C2_0_PIT_INPUT

C2_1_PIT_INPUT

No mezclar sus universos ni hashes.

C0/C1 continúan consumiendo sus fuentes actuales.

C2 sombra consumirá posteriormente únicamente snapshots PIT completos y
compatibles con su variante.

No conectar todavía C2 al archivo PIT hasta cerrar:

- contrato de factores;
- metodología sellada;
- methodology_hash;
- score continuo.

==================================================
9. TESTS
==================================================

Probar como mínimo:

1. snapshot completo;
2. retry idempotente;
3. conflicto de contenido;
4. ticker ausente;
5. factor ausente;
6. source failure;
7. rate limit;
8. universo parcial;
9. dos cutoffs incompatibles;
10. acceptedDate posterior al cutoff;
11. cache mtime distinto de acceptedDate;
12. contenido WRITE_ONCE correctamente capturado como AS_OBSERVED;
13. PIT no depende de fund_*.json productivo;
14. C0/C1 no cambian;
15. selección activa no cambia;
16. cero recomendaciones;
17. cero movimientos económicos;
18. legacy golden intacto;
19. compliant permanece READY_NOT_ACTIVE.

==================================================
10. TESTS PREEXISTENTES
==================================================

Documentar los cinco fallos preexistentes de:

growth-daily-selection-cycle

Entregar:

PREEXISTING_FAILURES =
<lista exacta>

BASELINE_FAILURE_OUTPUT_HASH =
<hash>

CURRENT_FAILURE_OUTPUT_HASH =
<mismo hash>

NEW_FAILURES_INTRODUCED =
0

No corregir esos tests dentro de este alcance.

==================================================
11. SESIONES DORADAS
==================================================

Mantener:

CURRENT_GOLDEN_SESSIONS =
8

PROMOTION_TARGET_GOLDEN_SESSIONS =
10

No reducir el criterio final.

El motor puede continuar en desarrollo y sombra con ocho sesiones.

La promoción permanece bloqueada hasta:

- alcanzar diez sesiones reproducibles;
- o recibir una excepción expresa posterior.

==================================================
12. COMMITS
==================================================

Crear commits separados:

1.
docs(growth): add normative C2 selection specification

2.
feat(growth): add as-observed selection input snapshots

3.
feat(growth): add append-only forward PIT archive

4.
test(growth): verify C2 input snapshot integrity

No mezclar en estos commits:

- score continuo;
- C2.0;
- C2.1;
- enriquecimiento completo;
- refresh productivo;
- activación del motor conforme.

No hacer push.
No crear tag.

==================================================
13. CHECKPOINT
==================================================

Entregar:

NORMATIVE_SPEC_PRESENT =
PASS/FAIL

NORMATIVE_SPEC_CONSISTENT =
PASS/FAIL

AS_OBSERVED_IMPLEMENTED =
PASS/FAIL

AS_OBSERVED_FIRST_SNAPSHOT_STATUS =
LAB_ONLY/NOT_CAPTURED

AS_OBSERVED_USES_CURRENT_C0_C1_INPUTS =
PASS/FAIL

PIT_FORWARD_ARCHIVE_IMPLEMENTED =
PASS/FAIL

PIT_FORWARD_USES_INDEPENDENT_STAGING =
PASS/FAIL

PIT_FORWARD_DEPENDS_ON_PRODUCTIVE_FUND_CACHE =
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

C0_C1_UNCHANGED =
PASS/FAIL

LEGACY_GOLDEN_HASH_UNCHANGED =
PASS/FAIL

CURRENT_GOLDEN_SESSIONS =
8

COMPLIANT_ENGINE_STATUS =
READY_NOT_ACTIVE

CONTINUOUS_FACTOR_CONTRACT =
PENDING/COMPLETE

C2_0_STATUS =
PLANNED

C2_1_STATUS =
INCOMPLETE_INPUT

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

No implementar todavía el score continuo si el contrato normativo de factores
sigue incompleto.

No desplegar ni activar nada.