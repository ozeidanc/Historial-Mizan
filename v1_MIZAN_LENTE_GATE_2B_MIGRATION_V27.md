MIZAN · LA LENTE
GATE 2B · MIGRACIÓN v27 ADITIVA Y VACÍA
SCHEMA POINT-IN-TIME SIN BACKFILL NI ACTIVACIÓN FUNCIONAL

NATURALEZA

Esta ejecución autoriza exclusivamente:

1. revisar y finalizar el DDL v27;
2. probar la migración en una copia LAB;
3. probar idempotencia y rollback vacío;
4. realizar backup de Producción;
5. aplicar la migración aditiva v27;
6. insertar únicamente definiciones metodológicas y de políticas;
7. verificar que las tablas operativas continúan vacías;
8. crear un único commit local de migración.

No autoriza:

- fetch de precios;
- backfill;
- creación de posiciones globales;
- creación de membership episodes;
- creación de allocation requests;
- creación de composiciones históricas;
- creación de transacciones papel;
- creación de flujos por cartera;
- creación de snapshots papel;
- reconstrucción del Track;
- cambio de daily-close;
- cambio de endpoints funcionales;
- cambio de UI;
- activación de TARGET_NOTIONAL_PRO_RATA;
- modificación de holdings históricos;
- modificación de catalizadores;
- deploy funcional;
- tag;
- push;
- operaciones remotas.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = f30aa3b
SCHEMA_BEFORE = v26
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Gate 2A cerrado con:

GLOBAL_PAPER_POSITIONS = 82
CURRENT_OPEN_GLOBAL_POSITIONS = 82

PORTFOLIO_MEMBERSHIP_EPISODES_PROPOSED = 121
CURRENT_OPEN_MEMBERSHIPS = 82
CLOSED_MEMBERSHIPS_BY_REASSIGNMENT = 39
ECONOMICALLY_CLOSED_GLOBAL_POSITIONS = 0
INTER_PORTFOLIO_REASSIGNMENTS = 39

REQUESTED_NOTIONAL_KNOWN = 46
REQUESTED_NOTIONAL_UNKNOWN = 36
REQUESTED_UNKNOWN_PRE_LOG = 26
REQUESTED_UNKNOWN_UNLINKED = 10

Metodología aprobada:

RETURN_METHOD =
PRICE_RETURN_TWR

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE_AS_OPERATED

PROSPECTIVE_ALLOCATION_POLICY =
TARGET_NOTIONAL_PRO_RATA_CONFIRMED

EXIT_POLICY =
KEEP_AS_PAPER_CASH

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

COMMISSION_POLICY =
NOT_MODELED

BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

La política prospectiva está aprobada, pero no activada.

==================================================
1. DECISIONES SELLADAS PARA EL SCHEMA
==================================================

A. REQUESTED NOTIONAL DESCONOCIDO

Los 36 casos sin solicitud recuperable deben soportarse como:

requested_notional = NULL

requested_notional_status:

- KNOWN;
- UNKNOWN_PRE_LOG;
- UNKNOWN_UNLINKED.

No utilizar valores ficticios o inferidos como solicitud de Omar.

B. RECONSTRUCCIÓN PRE-LOG

Los 26 casos pre-log pueden contener una reconstrucción candidata, pero su
estado inicial será:

reconstruction_confidence = INFERRED

certification_state =
MANUAL_REVIEW_REQUIRED

No podrán marcarse:

- CERTIFIED;
- AUTO_BACKFILL_SAFE;
- DETERMINISTIC;

sin evidencia adicional o aprobación manual.

El schema debe separar:

- valor reconstruido;
- fuente;
- confianza;
- estado de certificación.

C. POLÍTICA PROSPECTIVA

TARGET_NOTIONAL_PRO_RATA debe insertarse como:

status = APPROVED_PENDING_ACTIVATION

effective_from = NULL

No se activa por:

- aplicar la migración;
- arrancar el servidor;
- llegar una nueva sesión bursátil;
- ejecutar daily-close.

La activación exigirá una operación futura explícita después de que estén
verificados motor, preview, confirmación y UI.

D. PRECIOS

No obtener precios en este Gate.

Los 80 anchors pendientes y las 39 reasignaciones quedan para Gate 2C.

==================================================
2. PRECONDICIONES
==================================================

Antes de modificar:

- confirmar HEAD;
- confirmar env-info;
- confirmar una sola raíz Git;
- inventariar todos los archivos untracked de Gate 2A;
- confirmar schema v26;
- ejecutar SQLite integrity_check;
- confirmar una sola instancia;
- confirmar health;
- capturar contadores;
- capturar hashes del Track de Crecimiento;
- crear backup SQLite;
- calcular SHA-256 del backup;
- validar el backup mediante apertura e integrity_check.

Capturar al menos:

- movimientos;
- cash_events;
- baselines;
- decisiones;
- resizing;
- holdings;
- snapshots;
- valuations;
- tesis;
- catalizadores;
- cat_composicion_log;
- carteras_papel.

No continuar si:

- integrity_check falla;
- el backup no puede abrirse;
- existen escrituras concurrentes no controladas;
- el schema real no es v26;
- HEAD o env-info no coinciden.

==================================================
3. REVISIÓN FINAL DEL DDL
==================================================

Revisar:

scratchpad/gate2a/v27-bis-ddl-topology.sql

La propuesta menciona 11 tablas.

Antes de aplicar, entregar la lista exacta de tablas y la responsabilidad de
cada una.

Como mínimo, el modelo debe cubrir:

1. paper_track_methodology

Reglas económicas y temporales:

- return method;
- anchor policy;
- price basis;
- dividend policy;
- commission policy;
- benchmark policy;
- market timezone.

No contiene capital universal.

2. paper_allocation_policy

Definiciones versionadas de:

- LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE;
- TARGET_NOTIONAL_PRO_RATA_CONFIRMED;
- MANUAL_NOTIONAL_HARD_CAP, si se conserva como alternativa.

La política prospectiva se inserta sin effective_from y sin activación.

3. paper_portfolio_track_config

Configuración versionada por cartera:

- configured_capital;
- currency;
- methodology_id;
- allocation_policy_id;
- leverage_allowed;
- partial allocation policy;
- effective_from;
- effective_to;
- supersedes_config_id;
- status.

No insertar todavía configuraciones reales de las cinco carteras.

4. paper_global_position

Continuidad económica global de la posición papel.

5. paper_portfolio_membership_episode

Intervalo de pertenencia de la posición a una cartera.

Debe soportar:

- origen;
- destino;
- entrada;
- salida de composición;
- reasignación;
- continuidad global.

6. paper_allocation_request

Debe permitir:

- requested_notional nullable;
- requested_notional_status;
- referencia al log;
- estado de resolución;
- fuente.

7. paper_composition_version

Versión point-in-time de la composición por cartera.

8. paper_composition_member

Asignación de cada membership dentro de una composición:

- requested;
- allocated;
- pesos;
- cantidad antes/después;
- confianza;
- certificación.

9. paper_portfolio_flow

Únicamente flujos externos:

- initial contribution;
- contribution;
- withdrawal;
- correction explícita.

10. paper_portfolio_transaction

Operaciones internas:

- purchase;
- sale;
- partial sale;
- legacy rebalance buy/sell;
- pro-rata rebalance buy/sell;
- dividend;
- commission;
- split adjustment.

11. paper_track_snapshot

Serie point-in-time unitizada, benchmark y cobertura.

Si el DDL contiene más o menos tablas:

- explicar la diferencia;
- demostrar que no se duplican responsabilidades;
- demostrar que conserva la trazabilidad requerida.

==================================================
4. CONTRATOS Y CONSTRAINTS
==================================================

Definir enums mediante CHECK constraints o tablas de referencia.

Estados mínimos:

REQUESTED_NOTIONAL_STATUS:

- KNOWN;
- UNKNOWN_PRE_LOG;
- UNKNOWN_UNLINKED.

RECONSTRUCTION_CONFIDENCE:

- PROVEN;
- DETERMINISTIC;
- INFERRED;
- UNKNOWN.

CERTIFICATION_STATE:

- PENDING;
- MANUAL_REVIEW_REQUIRED;
- PRICE_COVERAGE_INCOMPLETE;
- CERTIFIED;
- INVALIDATED;
- SUPERSEDED.

METHODOLOGY_STATUS:

- DRAFT;
- SEALED;
- RETIRED.

ALLOCATION_POLICY_STATUS:

- DRAFT;
- APPROVED_PENDING_ACTIVATION;
- ACTIVE;
- RETIRED.

MEMBERSHIP_STATUS:

- OPEN;
- CLOSED;
- INVALIDATED;
- ARCHIVED.

GLOBAL_POSITION_STATUS:

- OPEN;
- CLOSED;
- INVALIDATED;
- ARCHIVED.

Reglas obligatorias:

- requested_notional puede ser NULL únicamente cuando su status lo permita;
- KNOWN exige requested_notional > 0;
- UNKNOWN_PRE_LOG y UNKNOWN_UNLINKED exigen requested_notional IS NULL;
- allocated_notional nunca se presenta como requested_notional;
- effective_to no puede preceder a effective_from;
- exited_at no puede preceder a entered_at;
- una composición no se modifica in-place después de certificarse;
- una metodología SEALED no se modifica in-place;
- un policy ACTIVE o RETIRED no se modifica in-place;
- flujos externos y transacciones internas no se mezclan;
- una reasignación no crea flujo externo;
- no existen importes negativos salvo tipos que los permitan expresamente;
- moneda explícita;
- timestamps en UTC;
- sesiones bursátiles en formato de fecha;
- hashes y claves de idempotencia obligatorios cuando corresponda.

==================================================
5. IDENTIDAD E IDEMPOTENCIA
==================================================

Definir claves únicas para impedir duplicados.

Como mínimo:

paper_track_methodology:

- methodology_id;
- version;
- methodology_hash único.

paper_allocation_policy:

- policy_id;
- version;
- policy_hash único.

paper_portfolio_track_config:

- paper_portfolio_id;
- effective_from;
- configuration_version.

paper_global_position:

- source_holding_id o una identidad legacy estable;
- no depender únicamente del ticker.

paper_portfolio_membership_episode:

- global_position_id;
- paper_portfolio_id;
- entered_at_utc.

paper_allocation_request:

- source_log_event_id cuando exista;
- idempotency_key alternativa para pre-log.

paper_composition_version:

- paper_portfolio_id;
- version;
- effective_at.

paper_composition_member:

- composition_version_id;
- membership_episode_id.

paper_portfolio_flow:

- idempotency_key.

paper_portfolio_transaction:

- idempotency_key.

paper_track_snapshot:

- paper_portfolio_id;
- methodology_id;
- valuation_session;
- methodology_version.

No crear una unicidad que impida:

- reincorporaciones;
- múltiples memberships de una posición global;
- reasignaciones sucesivas;
- nuevas versiones de composición;
- nuevas versiones metodológicas.

==================================================
6. FOREIGN KEYS Y CICLOS
==================================================

Activar y probar:

PRAGMA foreign_keys = ON

Definir de forma explícita:

- ON DELETE;
- ON UPDATE;
- deferrable, si es necesario.

Preferencia productiva:

- RESTRICT o NO ACTION para historia certificada;
- no utilizar CASCADE que pueda borrar historia financiera completa;
- SET NULL solo para referencias opcionales justificadas.

Las relaciones origen/destino de una reasignación pueden generar referencias
circulares.

Resolverlas mediante:

- inserción diferida;
- tabla de relación de reasignación;
- o foreign keys nullable actualizadas dentro de una transacción.

Documentar la solución.

No desactivar foreign keys para que la migración pase.

==================================================
7. MIGRACIÓN TRANSACCIONAL
==================================================

La migración v26 → v27 debe ejecutarse dentro de una transacción controlada.

Orden esperado:

1. BEGIN IMMEDIATE;
2. comprobar user_version = 26;
3. crear tablas de referencia o definiciones;
4. crear tablas operativas;
5. crear índices;
6. insertar metodología;
7. insertar políticas;
8. validar inserts;
9. establecer user_version = 27;
10. ejecutar comprobaciones estructurales;
11. COMMIT.

Si falla cualquier paso:

- ROLLBACK;
- schema debe permanecer v26;
- no dejar tablas parciales;
- no modificar user_version;
- no continuar.

La migración debe ser compatible con el mecanismo existente de migraciones
de Mizan.

No crear un mecanismo paralelo.

==================================================
8. FILAS PERMITIDAS EN GATE 2B
==================================================

Las únicas filas permitidas son definiciones globales inmutables.

A. METODOLOGÍA

Insertar:

methodology_id =
PAPER_PRICE_RETURN_TWR_V1

return_method =
PRICE_RETURN_TWR

anchor_policy =
OFFICIAL_CLOSE_POINT_IN_TIME

price_basis =
OFFICIAL_CLOSE_NOMINAL

dividend_policy =
INCOME_NOT_INCLUDED

commission_policy =
NOT_MODELED

benchmark_policy =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

market_timezone =
America/New_York

status =
SEALED

B. POLÍTICA HISTÓRICA

Insertar:

policy_id =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

status =
RETIRED_FOR_NEW_COMPOSITIONS

Debe seguir disponible para reconstrucción histórica.

C. POLÍTICA PROSPECTIVA

Insertar:

policy_id =
TARGET_NOTIONAL_PRO_RATA_CONFIRMED

status =
APPROVED_PENDING_ACTIVATION

effective_from =
NULL

confirmation_required =
true

leverage_allowed =
false

implicit_contribution_allowed =
false

negative_cash_allowed =
false

D. POLÍTICA ALTERNATIVA

Puede insertarse:

MANUAL_NOTIONAL_HARD_CAP

status =
AVAILABLE_NOT_DEFAULT

Solo si los estados del schema permiten distinguir una política disponible
de una activa.

No insertar en este Gate:

- configuraciones de las cinco carteras;
- global positions;
- memberships;
- allocation requests;
- composition versions;
- composition members;
- flows;
- transactions;
- track snapshots.

==================================================
9. PRUEBA EN LAB
==================================================

Aplicar primero v27 a una copia LAB validada de v26.

Ejecutar:

- migración inicial;
- integrity_check;
- foreign_key_check;
- inspección de schema;
- inspección de índices;
- inspección de constraints;
- inspección de filas metodológicas;
- comprobación de tablas operativas vacías.

Probar segunda ejecución.

Resultado permitido:

- no duplica tablas;
- no duplica definiciones;
- no cambia hashes;
- no crea filas operativas;
- mantiene user_version = 27;
- termina limpiamente.

La segunda ejecución no debe intentar aplicar nuevamente lógica destructiva.

==================================================
10. ROLLBACK VACÍO EN LAB
==================================================

Con las tablas operativas todavía vacías:

- probar rollback v27 → v26 en LAB;
- eliminar exclusivamente estructuras v27;
- restaurar user_version = 26;
- conservar todos los datos v26;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- comparar contadores y hashes.

Después volver a aplicar v27 en LAB para demostrar repetibilidad.

El rollback destructivo por DROP solo es válido mientras no existan:

- posiciones globales;
- memberships;
- solicitudes;
- composiciones;
- flujos;
- transacciones;
- snapshots.

Después de Gate 2C:

- rollback productivo = forward corrective migration;
- desactivar feature;
- conservar historia.

Documentarlo.

==================================================
11. PRUEBAS AUTOMÁTICAS
==================================================

Crear:

verify-paper-track-methodology-v27.mjs
verify-paper-track-migration-v27.mjs
verify-paper-track-schema-v27.mjs

Validar al menos:

- metodología SEALED;
- metodología sin capital universal;
- capital pertenece a configuración por cartera;
- legacy disponible para replay;
- pro-rata pendiente de activación;
- effective_from prospectivo NULL;
- requested_notional nullable con status;
- UNKNOWN no permite valor fingido;
- KNOWN exige valor;
- membership separado de global position;
- composición versionada;
- flujo externo separado de transacción interna;
- reasignación no es flujo externo;
- pre-log INFERRED soportado;
- certification_state soportado;
- foreign keys;
- uniqueness;
- idempotencia;
- tablas operativas vacías;
- rollback vacío;
- schema v27;
- cero cambios v26.

Requisitos por suite:

PASS > 0
FAIL = 0
NO_RESULT = 0

==================================================
12. APLICACIÓN EN PRODUCCIÓN
==================================================

Solo después de LAB completamente verde.

Antes:

- detener escrituras de la aplicación;
- confirmar una sola instancia;
- backup validado;
- SHA-256;
- integrity_check;
- foreign_key_check;
- contadores;
- hashes;
- schema v26.

Aplicar exclusivamente la migración.

Después:

- schema v27;
- reiniciar una sola instancia;
- health;
- env-info;
- integrity_check;
- foreign_key_check;
- contadores v26 idénticos;
- hashes de Crecimiento idénticos;
- definiciones metodológicas presentes;
- tablas operativas vacías.

Contadores obligatorios:

paper_global_position = 0
paper_portfolio_membership_episode = 0
paper_allocation_request = 0
paper_composition_version = 0
paper_composition_member = 0
paper_portfolio_flow = 0
paper_portfolio_transaction = 0
paper_track_snapshot = 0
paper_portfolio_track_config = 0

Las tablas de metodología y políticas pueden contener únicamente las filas
autorizadas.

==================================================
13. CÓDIGO Y COMMIT LOCAL
==================================================

Añadir al repositorio únicamente:

- migración v27;
- integración con el runner de migraciones existente;
- pruebas v27;
- documentación técnica de rollback;
- DDL final necesario.

No añadir automáticamente:

- scratchpad;
- previews JSON;
- copias de la base;
- backups;
- artefactos con datos productivos;
- archivos temporales;
- antiguos backups HTML.

Ejecutar secret scan sobre los archivos a incluir.

Crear un único commit local:

feat(lens): add empty v27 schema for point-in-time paper track

No crear tag.

No hacer push.

==================================================
14. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = <hash>
CANONICAL_HEAD_AFTER = <hash>

SCHEMA_BEFORE = v26
SCHEMA_AFTER = v27

V27_TABLES_CREATED = <lista>
V27_TABLE_COUNT = <número>
V27_INDEX_COUNT = <número>
V27_FOREIGN_KEY_COUNT = <número>

PAPER_TRACK_METHODOLOGY_ID =
PAPER_PRICE_RETURN_TWR_V1

PAPER_TRACK_METHODOLOGY_STATUS =
SEALED

HISTORICAL_POLICY_ID =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

HISTORICAL_POLICY_STATUS =
RETIRED_FOR_NEW_COMPOSITIONS

PROSPECTIVE_POLICY_ID =
TARGET_NOTIONAL_PRO_RATA_CONFIRMED

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

REQUESTED_UNKNOWN_SUPPORTED = PASS/FAIL
PRE_LOG_INFERRED_SUPPORTED = PASS/FAIL
GLOBAL_POSITION_MEMBERSHIP_SEPARATED = PASS/FAIL
COMPOSITION_VERSIONING_SUPPORTED = PASS/FAIL
EXTERNAL_INTERNAL_EVENTS_SEPARATED = PASS/FAIL

MIGRATION_TRANSACTIONAL = PASS/FAIL
MIGRATION_IDEMPOTENT = PASS/FAIL
EMPTY_ROLLBACK_VALIDATED_IN_LAB = PASS/FAIL
POST_DATA_ROLLBACK_DOCUMENTED = PASS/FAIL

PRODUCTION_OPERATIONAL_TABLES_EMPTY = PASS/FAIL
NO_BACKFILL_EXECUTED = PASS/FAIL
NO_PRICE_FETCH_PERFORMED = PASS/FAIL
NO_POLICY_ACTIVATED = PASS/FAIL
NO_PAPER_HISTORY_MUTATED = PASS/FAIL
NO_GROWTH_TRACK_CHANGED = PASS/FAIL
NO_REAL_DATA_MUTATED = PASS/FAIL
NO_THESES_CHANGED = PASS/FAIL
NO_CATALYSTS_CHANGED = PASS/FAIL
NO_HUNTING_FIELD_CHANGED = PASS/FAIL
NO_ECONOMIC_WRITES = PASS/FAIL
NO_REMOTE_OPERATIONS = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

LOCAL_COMMIT =
<hash>

Detenerse después del Gate 2B.