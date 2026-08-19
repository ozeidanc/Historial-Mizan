MIZAN · CHECKPOINT OPERATIVO URGENTE PREVIO A FASE 1 C2

La Fase 0 queda aprobada.

Este checkpoint se ejecuta antes de comenzar a implementar el score continuo
de C2.

No sustituye las restricciones de la especificación normativa.

==================================================
1. RESPUESTA Q1 · FICHERO NORMATIVO
==================================================

El contenido íntegro de:

spec-mizan-c2-seleccion.md

precede a esta orden y debe añadirse al repositorio mediante el commit
documental previsto.

La ejecución correcta de la Fase 0 sin el fichero es aceptada, pero no autoriza
la Fase 1 sin aplicar íntegramente:

- PARTE A · Diagnóstico;
- PARTE B · Restricciones de arquitectura.

Antes de continuar declarar:

NORMATIVE_SPEC_PRESENT =
PASS

NORMATIVE_SPEC_REVIEWED =
PASS

PHASE_0_CONSISTENT_WITH_NORMATIVE_SPEC =
PASS/FAIL

Si aparece una contradicción, detenerse y documentarla.

==================================================
2. RESPUESTA Q2 · PIT Y SOMBRA FORWARD
==================================================

Autorizo ambas rutas:

PRIMARY_VALIDATION_ROUTE =
ROUTE_5_FORWARD_SHADOW_ONLY

PARALLEL_DATA_ROUTE =
ROUTE_4_ARCHIVE_PIT_FROM_NOW_FORWARD

El archivo PIT debe comenzar cuanto antes porque cada sesión no archivada es
historia causal que no podrá reconstruirse después.

No existe una disyuntiva entre:

- archivar inputs PIT;
- ejecutar C2 en sombra.

Ambas actividades deben avanzar en paralelo.

El archivo PIT debe implementarse primero y debe ser:

- puramente aditivo;
- append-only;
- idempotente;
- sellado por sesión;
- reproducible;
- sin recomendaciones ejecutables;
- sin cambios económicos;
- sin modificar C0/C1;
- sin reutilizar C0_SELECTION_FROZEN;
- sin depender únicamente de universe-cache.json mutable.

Antes de activarlo en Producción:

- probarlo en copia/LAB;
- probar mismo input;
- probar conflicto de contenido;
- probar reinicio;
- probar scheduler;
- demostrar cero modificaciones en Book, cash, NAV, holdings,
  recomendaciones y movimientos.

No esperar a completar C2 para activar el archivo PIT.

==================================================
3. RESPUESTA Q3 · CADENCIA C0/C1
==================================================

No acepto automáticamente que la contradicción de cadencia deba permanecer
durante los 3–6 meses de sombra.

Antes de decidir, realizar una auditoría read-only y responder:

A. ALCANCE DEL METHODOLOGY HASH

Determinar exactamente qué serializa y protege:

methodology_hash =
103b4256f26d9e94c7c64730cd9e04c2953119962d2e246b2aef9d8302854182

Entregar:

HASHED_FIELDS =
<lista exacta>

HASH_INCLUDES_SELECTION_GATES =
YES/NO

HASH_INCLUDES_SORT_KEYS =
YES/NO

HASH_INCLUDES_TOP_N =
YES/NO

HASH_INCLUDES_REVIEW_FREQUENCY =
YES/NO

HASH_INCLUDES_FREEZE_CADENCE =
YES/NO

HASH_INCLUDES_ACTIVE_SELECTION_TRANSITION_RULES =
YES/NO

No inferirlo a partir del nombre del archivo: reproducir la construcción del
hash y enumerar el payload exacto.

B. CONTRATO SELLADO

Trazar todas las fuentes normativas de la cadencia:

- archivo sellado;
- passport;
- configuración;
- scheduler;
- documentación operativa;
- tests;
- audit trail.

Determinar qué significa exactamente:

scheduled_review_frequency =
trimestral

Distinguir:

- cálculo diario;
- registro diario de computed selection;
- revisión diaria de sizing;
- activación de nueva composición;
- rebalanceo trimestral;
- hard-risk exit;
- override manual.

Entregar:

SEALED_POLICY_REQUIRES_QUARTERLY_ACTIVE_COMPOSITION =
YES/NO/AMBIGUOUS

NIGHTLY_COMPUTATION_ALLOWED =
YES/NO/AMBIGUOUS

NIGHTLY_ACTIVE_COMPOSITION_REPLACEMENT_ALLOWED =
YES/NO/AMBIGUOUS

C. DECISIÓN CONDICIONAL

Si se cumplen simultáneamente:

1. el hash no incluye la cadencia ni la transición de active selection;
2. el sello exige inequívocamente composición activa trimestral;
3. el cálculo nocturno puede seguir existiendo como observación;
4. no existe otro contrato que autorice reemplazo diario;
5. el cambio puede probarse sin reescribir historia;

clasificar:

CADENCE_DEFECT =
CONFIRMED_RUNTIME_BUG

En ese caso proponer un hotfix separado:

computed_selection =
recalculada y auditada diariamente

active_selection =
congelada hasta el rebalanceo trimestral

No modificar:

- gates;
- score;
- ranking;
- top 25;
- metodología sellada;
- methodology_hash;
- selecciones históricas;
- recomendaciones ejecutadas;
- movimientos;
- holdings.

Si el hash sí incluye la cadencia, el contrato es ambiguo o el cambio altera la
semántica sellada, no aplicar el hotfix bajo C0/C1.

Clasificar entonces:

CADENCE_CHANGE_REQUIRES_NEW_METHODOLOGY =
YES

y mantener la corrección exclusivamente en C2.

==================================================
4. SEGURIDAD DEL POSIBLE HOTFIX DE CADENCIA
==================================================

El diagnóstico puede realizarse en:

feature/c2-continuous-scoring

Pero cualquier hotfix de C0/C1 debe implementarse en una rama separada desde
el HEAD productivo aplicable:

hotfix/growth-quarterly-active-selection

No mezclarlo con los commits de C2.

Antes de aplicar el hotfix:

- verificar el estado actual de la revisión C1;
- identificar la active selection vigente;
- identificar la próxima fecha trimestral;
- identificar computed selections posteriores;
- comparar qué recomendaciones cambiarían;
- comprobar fills pendientes;
- comprobar formularios abiertos;
- comprobar operaciones ejecutadas aún no registradas;
- ejecutar test dorado;
- realizar preview sin escrituras.

El hotfix no puede:

- activar una selección distinta durante el despliegue;
- generar recomendaciones automáticamente;
- superseder una revisión vigente por el despliegue;
- modificar operaciones;
- enviar órdenes a Wio.

La selección activa inmediatamente posterior al hotfix debe ser la selección
canónica vigente antes del cutover, salvo autorización expresa diferente.

No elegir retroactivamente otra selección nocturna.

==================================================
5. RESPUESTA Q4 · UNIVERSE-CACHE.JSON
==================================================

Auditar urgentemente por qué:

universe-cache.json

presenta:

mtime =
2026-06-21

Fecha actual de referencia:

2026-07-28

No tratarlo únicamente como limitación de backtesting.

Determinar si representa un defecto operativo de Producción.

Trazar:

1. proceso que crea el archivo;
2. proceso que debe actualizarlo;
3. frecuencia configurada;
4. scheduler;
5. startup catch-up;
6. endpoint o job de refresh;
7. fuente de datos;
8. credenciales y permisos;
9. rate limits;
10. errores registrados;
11. atomic rename/escritura;
12. validación del contenido;
13. fallback utilizado;
14. consumidores reales;
15. otras cachés o fuentes que puedan prevalecer;
16. alertas por staleness;
17. comportamiento cuando el archivo está vencido.

Entregar:

UNIVERSE_CACHE_PATH =
<ruta>

UNIVERSE_CACHE_MTIME =
2026-06-21T...

EXPECTED_REFRESH_FREQUENCY =
<frecuencia>

LAST_SUCCESSFUL_REFRESH =
<fecha>

LAST_ATTEMPTED_REFRESH =
<fecha/UNKNOWN>

REFRESH_JOB_ENABLED =
YES/NO

REFRESH_JOB_RUNNING =
YES/NO

REFRESH_FAILURES =
<lista/NONE>

CACHE_CONSUMED_BY_ACTIVE_C0_C1 =
YES/NO/PARTIAL

ACTIVE_MODEL_LATEST_FUNDAMENTAL_AS_OF =
<fecha>

NEWER_DATA_EXISTS_ELSEWHERE =
YES/NO

STALE_CACHE_FALLBACK_ACTIVE =
YES/NO

STALE_CACHE_ALERT_PRESENT =
YES/NO

ROOT_CAUSE =
<causa>

OPERATIONAL_IMPACT =
<impacto>

No concluir que C0/C1 llevan cinco semanas sin datos nuevos hasta confirmar
que el cache es la fuente efectiva consumida por el motor.

==================================================
6. CLASIFICACIÓN DEL CACHE
==================================================

Clasificar como uno de:

A. EXPECTED_IMMUTABLE_REVIEW_SNAPSHOT

El archivo fue diseñado para permanecer congelado durante un periodo concreto
y existe otra fuente actual para las revisiones nuevas.

B. REFRESH_JOB_FAILURE

Existe un proceso de actualización, pero no se ha ejecutado o ha fallado.

C. STALE_CACHE_SILENTLY_ACCEPTED

El motor consume datos vencidos sin alerta ni bloqueo.

D. SOURCE_DATA_NOT_UPDATED

El proceso funciona, pero la fuente no entrega datos nuevos.

E. CACHE_METADATA_MISLEADING

El mtime no representa correctamente la fecha de los datos consumidos.

F. UNKNOWN_BLOCKED

No puede determinarse con la evidencia disponible.

Si resulta B o C:

CACHE_OPERATIONAL_BUG =
CONFIRMED

y debe proponerse un hotfix separado antes de confiar en nuevas selecciones.

==================================================
7. POSIBLE HOTFIX DEL CACHE
==================================================

No refrescar automáticamente Producción durante el diagnóstico.

Primero ejecutar un preview que muestre:

- número de tickers;
- fechas de datos actuales;
- fechas de datos candidatos;
- factores modificados;
- scores binarios modificados;
- elegibilidad modificada;
- ranking modificado;
- top 25 actual;
- top 25 candidato;
- recomendaciones que produciría;
- diferencias de composición.

Un refresh de fundamentales puede cambiar la selección productiva.

Por tanto:

- no sustituir el cache en caliente;
- no ejecutar freeze;
- no publicar revisión;
- no generar recomendaciones;
- no modificar active selection.

Si el refresh es correcto, desplegar primero el mecanismo y sellar los datos
nuevos sin activarlos como composición productiva hasta resolver la cadencia.

Separar:

DATA_REFRESH =
obtención y archivo de datos nuevos

SELECTION_ACTIVATION =
decisión de cambiar la composición

Nunca fusionar ambos pasos.

==================================================
8. ORDEN DE PRIORIDAD
==================================================

El orden de trabajo queda:

1. incorporar y validar la especificación normativa;
2. auditar el alcance del methodology_hash y el contrato trimestral;
3. auditar universe-cache.json;
4. diseñar y probar el archivo PIT;
5. entregar este checkpoint;
6. si procede, autorizar hotfix del cache;
7. si procede, autorizar hotfix de cadencia en rama separada;
8. activar archivo PIT;
9. implementar C2 en sombra;
10. iniciar observación forward;
11. no promover durante 3–6 meses o hasta evidencia suficiente.

No iniciar todavía el score continuo antes de cerrar los puntos 1–5.

==================================================
9. ENTREGA DEL CHECKPOINT
==================================================

NORMATIVE_SPEC_PRESENT =
PASS/FAIL

NORMATIVE_SPEC_CONSISTENT_WITH_PHASE_0 =
PASS/FAIL

METHODOLOGY_HASH_PAYLOAD =
<resumen exacto>

HASH_INCLUDES_FREEZE_CADENCE =
YES/NO

SEALED_POLICY_REQUIRES_QUARTERLY_ACTIVE_COMPOSITION =
YES/NO/AMBIGUOUS

CADENCE_DEFECT =
CONFIRMED_RUNTIME_BUG /
NEW_METHODOLOGY_REQUIRED /
AMBIGUOUS

CADENCE_HOTFIX_RECOMMENDED =
YES/NO

UNIVERSE_CACHE_MTIME =
<timestamp>

EXPECTED_CACHE_REFRESH_FREQUENCY =
<frecuencia>

ACTIVE_MODEL_LATEST_FUNDAMENTAL_AS_OF =
<fecha>

CACHE_CONSUMED_BY_ACTIVE_C0_C1 =
YES/NO/PARTIAL

CACHE_CLASSIFICATION =
A/B/C/D/E/F

CACHE_OPERATIONAL_BUG =
CONFIRMED/NOT_CONFIRMED/UNKNOWN

CACHE_HOTFIX_RECOMMENDED =
YES/NO

PIT_ARCHIVE_DESIGN =
PASS/FAIL

PIT_ARCHIVE_CAN_START_IMMEDIATELY =
YES/NO

C0_FILE_MODIFIED =
NO

C0_METHODOLOGY_HASH_UNCHANGED =
PASS

C1_FILE_MODIFIED =
NO

FILES_MODIFIED =
0

PRODUCTION_WRITES =
0

EXACT_DECISIONS_REQUIRED_FROM_OMAR =
<lista/NONE>

CHECKPOINT_STATUS =
COMPLETE

Detenerse después de la entrega.

No implementar el score continuo.
No desplegar.
No refrescar datos productivos.
No cambiar la active selection.