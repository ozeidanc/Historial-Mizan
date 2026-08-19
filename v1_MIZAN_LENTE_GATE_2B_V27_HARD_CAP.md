MIZAN · LA LENTE
GATE 2B · MIGRACIÓN v27 ADITIVA Y VACÍA
METODOLOGÍA SELLADA · MANUAL NOTIONAL HARD CAP PENDIENTE DE ACTIVACIÓN

NATURALEZA

Esta ejecución autoriza exclusivamente:

1. finalizar el DDL v27;
2. probar la migración en LAB;
3. probar constraints, foreign keys e idempotencia;
4. probar rollback vacío en LAB;
5. crear y validar un backup de Producción;
6. aplicar la migración aditiva v27;
7. insertar únicamente definiciones metodológicas y políticas;
8. mantener vacías todas las tablas operativas;
9. crear un único commit local.

No autoriza:

- fetch de precios;
- backfill;
- creación de posiciones globales;
- creación de membership episodes;
- creación de solicitudes de asignación;
- creación de configuraciones para las cinco carteras;
- creación de composiciones;
- creación de flujos;
- creación de transacciones;
- creación de snapshots;
- cambio de daily-close;
- cambio de endpoints funcionales;
- cambio de UI;
- activación de la nueva política;
- modificación del comportamiento actual;
- deploy funcional del Track Papel;
- modificación de holdings;
- modificación de tesis;
- modificación de catalizadores;
- modificación de Campo de Caza;
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

Inventario histórico confirmado:

GLOBAL_PAPER_POSITIONS = 82
CURRENT_OPEN_GLOBAL_POSITIONS = 82

PORTFOLIO_MEMBERSHIP_EPISODES_PROPOSED = 121
CURRENT_OPEN_MEMBERSHIP_EPISODES = 82
CLOSED_MEMBERSHIP_EPISODES = 39
ECONOMICALLY_CLOSED_GLOBAL_POSITIONS = 0
INTER_PORTFOLIO_REASSIGNMENTS = 39
REINCORPORATIONS = 0

REQUESTED_NOTIONAL_KNOWN = 46
REQUESTED_NOTIONAL_UNKNOWN = 36
REQUESTED_UNKNOWN_PRE_LOG = 26
REQUESTED_UNKNOWN_UNLINKED = 10

Histórico confirmado:

HISTORICAL_TRACK_POLICY =
AS_OPERATED

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

Política futura aprobada:

PROSPECTIVE_ALLOCATION_POLICY =
MANUAL_NOTIONAL_HARD_CAP

PROSPECTIVE_AUTO_REBALANCE =
DISABLED

PARTIAL_ALLOCATION_POLICY =
REQUIRE_EXPLICIT_CONFIRMATION

LEVERAGE_POLICY =
DISABLED

Metodología aprobada:

RETURN_METHOD =
PRICE_RETURN_TWR

EXIT_POLICY =
KEEP_AS_PAPER_CASH

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

COMMISSION_POLICY =
NOT_MODELED

BENCHMARK =
SPY

BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

La política prospectiva está aprobada, pero no activada.

==================================================
1. CONTRATO DE LA POLÍTICA PROSPECTIVA
==================================================

La política MANUAL_NOTIONAL_HARD_CAP significa:

1. Cada cartera tiene un capital configurado propio.
2. Omar introduce requested_notional para cada stock.
3. requested_notional se conserva y no se sobrescribe.
4. La asignación se financia con paper_cash.
5. Las posiciones existentes no se reponderan automáticamente.
6. No existe normalización automática capital/N.
7. No existe ampliación implícita del capital.
8. No existe cash negativo.
9. No existe leverage implícito.
10. No existe rebalanceo automático.

Si:

paper_cash >= requested_notional

entonces:

allocated_notional = requested_notional
allocation_status = FULLY_ALLOCATED

La operación futura será una compra interna:

- paper_cash disminuye;
- paper_market_value aumenta;
- unidades TWR no cambian;
- no existe flujo externo;
- la compra no genera rentabilidad.

Si:

0 < paper_cash < requested_notional

entonces, por defecto:

allocated_notional = 0
allocation_status = PENDING_CAPITAL

No realizar asignación parcial automáticamente.

La UI futura deberá ofrecer opciones explícitas:

A. Mantener pendiente.
B. Asignar únicamente el cash disponible.
C. Crear una aportación externa.
D. Solicitar un rebalanceo explícito.
E. Cancelar la solicitud.

La opción B requiere confirmar:

- requested_notional;
- proposed_allocated_notional;
- diferencia no asignada;
- paper_cash resultante;
- peso resultante.

Si:

paper_cash = 0

entonces:

allocated_notional = 0
allocation_status = PENDING_CAPITAL

No crear:

- posición activa;
- membership episode efectivo;
- compra;
- exposición;
- cantidad certificada;
- punto de Track.

==================================================
2. REBALANCEO EXPLÍCITO
==================================================

PROSPECTIVE_AUTO_REBALANCE = DISABLED

Un rebalanceo solo puede ejecutarse mediante una instrucción expresa de Omar.

Antes de confirmar debe mostrarse un preview con:

- composición actual;
- composición propuesta;
- requested_notional;
- allocated_notional actual;
- allocated_notional propuesto;
- cantidades actuales;
- cantidades propuestas;
- pesos actuales;
- pesos propuestos;
- compras;
- ventas;
- turnover;
- paper cash antes;
- paper cash después;
- fecha y sesión efectiva;
- cobertura de precios;
- warnings.

El rebalanceo:

- crea una nueva composition version;
- crea transacciones internas;
- no crea flujo externo;
- no emite ni retira unidades TWR;
- no cambia anchors originales;
- no cambia incorporated_at;
- no reescribe composiciones anteriores;
- no modifica retroactivamente el Track.

La existencia futura de un rebalanceo explícito no convierte la política
prospectiva en auto-rebalance.

==================================================
3. REASIGNACIONES ENTRE CARTERAS
==================================================

Una reasignación prospectiva debe respetar el hard cap de la cartera destino.

La solicitud de mover una posición entre carteras debe generar primero un
preview independiente para:

A. CARTERA DE ORIGEN

- cerrar membership episode;
- vender o retirar la asignación de su composición;
- convertir el valor en paper_cash;
- mantener unidades TWR;
- no crear retirada externa.

B. CARTERA DE DESTINO

- comprobar requested_notional;
- comprobar paper_cash disponible;
- aplicar MANUAL_NOTIONAL_HARD_CAP.

Si la cartera destino tiene suficiente cash:

- crear nuevo membership episode;
- crear compra interna;
- allocated_notional = requested_notional.

Si no tiene suficiente cash:

- no completar automáticamente la reasignación;
- no sacar todavía la posición del origen salvo confirmación atómica;
- marcar la operación propuesta como PENDING_CAPITAL;
- ofrecer aportación, asignación parcial, rebalanceo o cancelación.

La reasignación debe ser atómica.

No permitir:

- posición eliminada del origen sin entrada válida en destino;
- duplicación accidental entre carteras;
- cash negativo en destino;
- aportación implícita;
- transferencia de unidades TWR entre carteras.

La historia anterior mantiene:

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

Las 39 reasignaciones históricas se reconstruirán AS_OPERATED en Gate 2C.

==================================================
4. SOLICITUDES PENDIENTES
==================================================

Una solicitud pendiente debe persistirse independientemente de un episodio
económico.

Estados mínimos:

- PENDING_CAPITAL;
- PENDING_PARTIAL_CONFIRMATION;
- PENDING_REBALANCE_CONFIRMATION;
- PENDING_EXTERNAL_CONTRIBUTION;
- FULLY_ALLOCATED;
- PARTIALLY_ALLOCATED;
- CANCELLED;
- REJECTED;
- SUPERSEDED.

Una solicitud pendiente:

- conserva requested_notional;
- no crea allocated_notional positivo;
- no crea global position económica activa;
- no crea membership efectivo;
- no entra en paper_market_value;
- no entra en gross exposure;
- no entra en Track;
- no entra en benchmark;
- no genera transacciones;
- no cambia unidades.

Si una asignación parcial es confirmada:

- allocated_notional contiene el importe aceptado;
- requested_notional conserva la solicitud original;
- allocation_status = PARTIALLY_ALLOCATED;
- la diferencia debe permanecer visible;
- no crear automáticamente una segunda solicitud por la diferencia.

==================================================
5. REQUESTED NOTIONAL DESCONOCIDO
==================================================

Los 36 registros históricos sin solicitud recuperable deben soportarse como:

requested_notional = NULL

requested_notional_status:

- UNKNOWN_PRE_LOG;
- UNKNOWN_UNLINKED.

Inventario:

REQUESTED_UNKNOWN_PRE_LOG = 26
REQUESTED_UNKNOWN_UNLINKED = 10

No utilizar como requested_notional:

- allocated_notional;
- capital/N;
- cantidad × precio;
- 1.000 por analogía;
- requested_notional de otro registro.

Para los 26 pre-log:

reconstruction_confidence =
INFERRED

certification_state =
MANUAL_REVIEW_REQUIRED

Puede conservarse una reconstrucción candidata de allocated_notional, pero no
certificarse automáticamente.

El schema debe separar:

- requested_notional;
- requested_notional_status;
- allocated_notional;
- allocation_source;
- reconstruction_confidence;
- certification_state.

==================================================
6. ACTIVACIÓN DE LA POLÍTICA PROSPECTIVA
==================================================

Insertar MANUAL_NOTIONAL_HARD_CAP como:

status =
APPROVED_PENDING_ACTIVATION

effective_from =
NULL

auto_rebalance =
false

partial_allocation_policy =
REQUIRE_EXPLICIT_CONFIRMATION

leverage_allowed =
false

implicit_contribution_allowed =
false

negative_cash_allowed =
false

La migración v27 no activa la política.

No activarla por:

- arranque del servidor;
- aplicación de la migración;
- llegada de una nueva sesión;
- daily-close;
- apertura de la UI;
- creación de una tesis.

La activación futura requiere:

1. motor de asignación implementado;
2. preview de capacidad;
3. confirmación explícita;
4. solicitudes pendientes soportadas;
5. reasignaciones atómicas soportadas;
6. UI validada;
7. tests verdes;
8. effective_from explícito.

La fecha se decidirá en el subgate funcional correspondiente.

==================================================
7. METODOLOGÍA HISTÓRICA
==================================================

Insertar la política histórica:

policy_id =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

status =
RETIRED_FOR_NEW_COMPOSITIONS

historical_replay_allowed =
true

new_allocation_allowed =
false

La política legacy debe permanecer disponible para reconstruir:

- composiciones históricas;
- reponderaciones capital/N;
- 39 reasignaciones;
- Track AS_OPERATED.

No debe utilizarse para nuevas solicitudes después de activar la política
prospectiva.

No convertir el histórico al hard cap.

No reinterpretar retrospectivamente:

- posición 11;
- posición 25;
- cantidades;
- pesos;
- asignaciones.

==================================================
8. METODOLOGÍA DEL TRACK
==================================================

Insertar:

methodology_id =
PAPER_PRICE_RETURN_TWR_V1

RETURN_METHOD =
PRICE_RETURN_TWR

ANCHOR_POLICY =
OFFICIAL_CLOSE_POINT_IN_TIME

PRICE_BASIS =
OFFICIAL_CLOSE_NOMINAL

MARKET_TIMEZONE =
America/New_York

EXIT_POLICY =
KEEP_AS_PAPER_CASH

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

COMMISSION_POLICY =
NOT_MODELED

BENCHMARK =
SPY

BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

status =
SEALED

La metodología no debe contener:

- capital universal;
- un importe fijo de 10.000;
- política activa global para todas las carteras;
- effective_from de la política prospectiva.

El capital y la política efectiva pertenecen a la configuración versionada de
cada cartera.

PRICE_RETURN_TWR debe mostrarse posteriormente con:

INCOME_NOT_INCLUDED

No llamarlo:

- total return;
- dividend-adjusted return;
- retorno total.

==================================================
9. CONFIGURACIÓN POR CARTERA
==================================================

El schema v27 debe permitir una configuración versionada por cartera:

- paper_portfolio_id;
- methodology_id;
- configured_capital;
- currency;
- allocation_policy_id;
- partial_allocation_policy;
- leverage_allowed;
- effective_from;
- effective_to;
- supersedes_config_id;
- status;
- configuration_hash;
- created_at.

No insertar configuraciones reales de las cinco carteras en Gate 2B.

El capital no debe guardarse en paper_track_methodology.

Cada cartera puede tener:

- capital diferente;
- política diferente;
- fecha de activación distinta.

No modificar una configuración histórica in-place.

==================================================
10. TABLAS v27
==================================================

Revisar y finalizar el DDL v27 para incluir como mínimo:

1. paper_track_methodology

Metodología del Track.

2. paper_allocation_policy

Políticas legacy, hard cap y alternativas versionadas.

3. paper_portfolio_track_config

Capital y política efectiva por cartera.

4. paper_global_position

Continuidad económica de la posición papel.

5. paper_portfolio_membership_episode

Intervalos de pertenencia por cartera.

6. paper_allocation_request

Solicitud manual y estado de capacidad.

Debe poder existir sin episodio efectivo.

7. paper_composition_version

Composición versionada.

8. paper_composition_member

Asignación dentro de cada composición.

9. paper_portfolio_flow

Solo flujos externos.

10. paper_portfolio_transaction

Compras, ventas y rebalanceos internos.

11. paper_track_snapshot

Serie unitizada y benchmark.

Si existen tablas adicionales:

- justificar su responsabilidad;
- demostrar que no duplican estado;
- incluirlas en el inventario final.

==================================================
11. CONSTRAINTS DE ASIGNACIÓN
==================================================

REQUESTED_NOTIONAL_STATUS:

- KNOWN;
- UNKNOWN_PRE_LOG;
- UNKNOWN_UNLINKED.

ALLOCATION_STATUS:

- PENDING_CAPITAL;
- PENDING_PARTIAL_CONFIRMATION;
- PENDING_REBALANCE_CONFIRMATION;
- PENDING_EXTERNAL_CONTRIBUTION;
- FULLY_ALLOCATED;
- PARTIALLY_ALLOCATED;
- CANCELLED;
- REJECTED;
- SUPERSEDED.

Reglas:

- KNOWN exige requested_notional > 0;
- UNKNOWN_* exige requested_notional IS NULL;
- PENDING_* exige allocated_notional = 0 o NULL;
- FULLY_ALLOCATED exige allocated_notional = requested_notional;
- PARTIALLY_ALLOCATED exige:
  - requested_notional > 0;
  - allocated_notional > 0;
  - allocated_notional < requested_notional;
  - confirmación explícita;
- CANCELLED y REJECTED no crean episodio efectivo;
- no se permite allocated_notional negativo;
- no se permite cash negativo;
- allocated_notional no puede superar requested_notional bajo hard cap;
- una asignación no crea flujo externo;
- una solicitud pendiente no crea transacción interna;
- un episodio efectivo exige allocated_notional > 0.

La comparación monetaria debe contemplar tolerancia explícita de redondeo.

No depender de igualdad binaria exacta de floats.

Preferir almacenamiento monetario seguro:

- minor units enteras; o
- decimal controlado según el patrón existente de Mizan.

Documentar la elección.

==================================================
12. FOREIGN KEYS E IDEMPOTENCIA
==================================================

Activar:

PRAGMA foreign_keys = ON

Preferir:

- RESTRICT;
- NO ACTION;
- SET NULL solo en referencias opcionales justificadas.

No usar CASCADE para borrar historia financiera.

Definir claves únicas que permitan:

- múltiples memberships por posición global;
- reincorporaciones;
- múltiples configuraciones;
- múltiples versiones de composición;
- políticas versionadas;
- metodologías futuras.

Claves de idempotencia obligatorias para:

- allocation requests;
- flows;
- transactions;
- composition versions;
- track snapshots.

Una reasignación debe tener una identidad estable compartida por origen y
destino.

No desactivar foreign keys durante migración o pruebas.

==================================================
13. FILAS PERMITIDAS EN GATE 2B
==================================================

Solo insertar definiciones globales.

A. METODOLOGÍA

PAPER_PRICE_RETURN_TWR_V1
status = SEALED

B. HISTÓRICA

LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE
status = RETIRED_FOR_NEW_COMPOSITIONS

C. PROSPECTIVA

MANUAL_NOTIONAL_HARD_CAP
status = APPROVED_PENDING_ACTIVATION
effective_from = NULL

D. OPCIONAL

TARGET_NOTIONAL_PRO_RATA_CONFIRMED

Puede conservarse como política disponible, no predeterminada:

status = AVAILABLE_NOT_DEFAULT

Solo si no introduce ambigüedad.

No insertar:

- configuraciones reales;
- posiciones;
- memberships;
- requests;
- composiciones;
- miembros;
- flows;
- transactions;
- snapshots.

==================================================
14. MIGRACIÓN TRANSACCIONAL
==================================================

Migrar v26 → v27 mediante el runner existente.

Orden:

1. BEGIN IMMEDIATE;
2. comprobar user_version = 26;
3. crear tablas;
4. crear índices;
5. crear constraints o tablas de referencia;
6. insertar definiciones globales;
7. validar definiciones;
8. establecer user_version = 27;
9. ejecutar comprobaciones;
10. COMMIT.

Si falla:

- ROLLBACK;
- mantener v26;
- no dejar tablas parciales;
- no modificar user_version;
- no continuar.

La segunda ejecución debe ser idempotente.

==================================================
15. LAB Y ROLLBACK VACÍO
==================================================

Antes de Producción:

- aplicar en copia LAB;
- integrity_check;
- foreign_key_check;
- inspeccionar tablas;
- inspeccionar índices;
- inspeccionar constraints;
- confirmar filas autorizadas;
- confirmar tablas operativas vacías.

Probar:

1. v26 → v27;
2. segunda ejecución;
3. v27 → v26 mientras está vacío;
4. integrity_check;
5. comparación de datos v26;
6. reaplicar v27.

Después de Gate 2C, no usar DROP TABLE como rollback productivo.

Utilizar migración correctiva hacia delante y conservar la historia.

==================================================
16. PRUEBAS AUTOMÁTICAS
==================================================

Crear:

verify-paper-track-methodology-v27.mjs
verify-paper-track-migration-v27.mjs
verify-paper-track-schema-v27.mjs
verify-paper-allocation-policy-v27.mjs

Validar:

- PRICE_RETURN_TWR;
- INCOME_NOT_INCLUDED;
- SPY;
- benchmark external-flows-only;
- legacy solo para replay;
- hard cap pendiente de activación;
- hard cap sin auto-rebalance;
- partial allocation con confirmación;
- leverage desactivado;
- cash negativo prohibido;
- requested_notional preservado;
- requested desconocido nullable;
- pending no crea episodio;
- fully allocated respeta notional;
- partial allocated requiere confirmación;
- config por cartera;
- capital fuera de la metodología global;
- global position separada de membership;
- composición versionada;
- flujos externos separados de transacciones;
- reasignación atómica modelable;
- idempotencia;
- foreign keys;
- tablas operativas vacías;
- rollback vacío;
- schema v27.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

==================================================
17. PRODUCCIÓN
==================================================

Antes:

- detener escrituras de la aplicación;
- una sola instancia;
- confirmar HEAD y env-info;
- backup SQLite;
- SHA-256;
- abrir backup;
- integrity_check del backup;
- foreign_key_check;
- capturar contadores;
- capturar hashes de Crecimiento;
- confirmar schema v26.

Aplicar únicamente la migración.

Después:

- schema v27;
- una sola instancia;
- health;
- env-info;
- integrity_check;
- foreign_key_check;
- contadores históricos idénticos;
- hashes de Crecimiento idénticos;
- definiciones globales presentes;
- tablas operativas vacías;
- política prospectiva no activa.

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

==================================================
18. CÓDIGO Y COMMIT
==================================================

Añadir únicamente:

- migración v27;
- integración con runner existente;
- DDL;
- pruebas;
- documentación de rollback.

No añadir:

- scratchpad;
- previews;
- bases;
- backups;
- JSON con datos productivos;
- artefactos temporales;
- backups HTML antiguos.

Ejecutar secret scan.

Crear un único commit local:

feat(lens): add empty v27 schema for manual paper allocation

No crear tag.

No hacer push.

==================================================
19. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = <hash>
CANONICAL_HEAD_AFTER = <hash>

SCHEMA_BEFORE = v26
SCHEMA_AFTER = v27

HISTORICAL_TRACK_POLICY =
AS_OPERATED

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

PROSPECTIVE_ALLOCATION_POLICY =
MANUAL_NOTIONAL_HARD_CAP

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

PROSPECTIVE_AUTO_REBALANCE =
DISABLED

PARTIAL_ALLOCATION_POLICY =
REQUIRE_EXPLICIT_CONFIRMATION

LEVERAGE_POLICY =
DISABLED

RETURN_METHOD =
PRICE_RETURN_TWR

EXIT_POLICY =
KEEP_AS_PAPER_CASH

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

COMMISSION_POLICY =
NOT_MODELED

BENCHMARK =
SPY

BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

V27_TABLES_CREATED = <lista>
V27_TABLE_COUNT = <número>
V27_INDEX_COUNT = <número>
V27_FOREIGN_KEY_COUNT = <número>

CAPITAL_MODELED_PER_PORTFOLIO = PASS/FAIL
REQUESTED_NOTIONAL_PRESERVED = PASS/FAIL
ALLOCATED_NOTIONAL_SEPARATED = PASS/FAIL
PENDING_REQUEST_WITHOUT_EPISODE_SUPPORTED = PASS/FAIL
PARTIAL_ALLOCATION_REQUIRES_CONFIRMATION = PASS/FAIL
AUTO_REBALANCE_DISABLED = PASS/FAIL
NEGATIVE_CASH_PROHIBITED = PASS/FAIL
LEVERAGE_DISABLED = PASS/FAIL
ATOMIC_REASSIGNMENT_MODELED = PASS/FAIL

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

LOCAL_COMMIT = <hash>

Detenerse después del Gate 2B.