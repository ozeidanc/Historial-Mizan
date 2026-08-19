MIZAN · LA LENTE
GATE 2F-A1 · RECUPERACIÓN DE SESIÓN Y CONTINUACIÓN CONTROLADA

Code se detuvo durante Gate 2F-A1.

No reiniciar el trabajo desde cero.
No descartar cambios.
No repetir operaciones ciegamente.
No aplicar nada en Producción.
No activar MANUAL_NOTIONAL_HARD_CAP.
No hacer push ni tag.

==================================================
1. AUDITAR EL ESTADO ACTUAL
==================================================

Baseline original esperado:

CANONICAL_HEAD_ORIGINAL = 0466595
PRODUCTION_SCHEMA = v29
TARGET_LAB_SCHEMA = v30

Determinar y reportar:

- HEAD actual;
- git status;
- archivos tracked modificados;
- archivos untracked;
- commits creados durante la sesión;
- migración v30 creada o incompleta;
- tablas v30 propuestas;
- servicios implementados;
- pruebas implementadas;
- pruebas ejecutadas;
- último paso completado;
- primer paso pendiente;
- errores o causa exacta de la detención.

Verificar Producción:

- schema sigue v29;
- integrity_check;
- foreign_key_check;
- política sigue APPROVED_PENDING_ACTIVATION;
- effective_from sigue NULL;
- datos productivos intactos;
- cero reservas productivas;
- cero acciones productivas;
- cero compras o ventas;
- cero snapshots nuevos por este gate.

No continuar si Producción fue modificada accidentalmente.
En ese caso detenerse y entregar diagnóstico exacto.

==================================================
2. CLASIFICAR EL ESTADO
==================================================

Clasificar:

A. NO_CHANGES

No hay cambios útiles.

B. PARTIAL_UNCOMMITTED

Existen cambios útiles sin commit.

C. PARTIAL_COMMITTED

Existe uno o más commits parciales.

D. IMPLEMENTATION_COMPLETE_TESTS_PENDING

Código aparentemente completo; faltan pruebas.

E. COMPLETE

Todo el alcance de 2F-A1 está terminado.

No crear un segundo conjunto paralelo de tablas o servicios.

Si ya existe una migración v30:

- inspeccionarla;
- continuar sobre ella;
- no crear otra migración v30.

Si ya existe un commit parcial:

- conservarlo;
- no resetearlo;
- continuar desde su HEAD;
- informar sus hashes.

==================================================
3. ALCANCE PRIORITARIO PARA TERMINAR
==================================================

Completar únicamente la fundación LAB de Gate 2F-A1:

1. migración v29 → v30;
2. lifecycle append-only;
3. reservas atómicas de Paper Cash;
4. reservas atómicas de cantidad;
5. cálculo de capacidad;
6. cálculo de cantidad disponible;
7. preview de nueva asignación;
8. preview de mantener/reducir/vender;
9. confirmación completa y parcial de compra;
10. confirmación de reducción y venta total;
11. decisión MANTENER;
12. cancelación y liberación;
13. concurrencia;
14. idempotencia;
15. reconciliación de alertas con posiciones papel;
16. pruebas.

No implementar todavía:

- endpoints públicos;
- UI;
- daily-close;
- catch-up;
- compras EOD;
- ventas EOD;
- composiciones prospectivas;
- memberships prospectivos;
- snapshots;
- carteras nuevas;
- aportaciones;
- activación de política.

==================================================
4. SI EL ALCANCE SIGUE SIENDO DEMASIADO GRANDE
==================================================

Dividir internamente sin detenerse para pedir permiso:

SUBGATE 2F-A1.1

- migración v30;
- lifecycle;
- cash reservations;
- capacity;
- confirm/cancel;
- concurrencia e idempotencia;
- posición 11.

SUBGATE 2F-A1.2

- position action lifecycle;
- quantity reservations;
- mantener/reducir/vender;
- concurrencia de cantidad;
- reconciliación de alertas.

Completar primero 2F-A1.1.

Si el contexto o tiempo no permite completar también 2F-A1.2:

- dejar 2F-A1.1 completamente verde;
- crear un commit local coherente;
- entregar checkpoint exacto;
- no dejar una migración o implementación parcialmente funcional.

==================================================
5. ALERTAS DE CATALIZADOR
==================================================

Reconciliar en LAB:

- ONB;
- ATRC;
- ISRG;
- HCSG;
- WSBC;
- WTFC;
- PEGA.

Para cada ticker resolver:

- estado del catalizador;
- global_position_id;
- membership económico abierto;
- cartera papel real;
- cantidad canónica;
- elegibilidad para mantener/reducir/vender;
- blocker, si existe.

No interpretar C1/C2/C3/C4/C6 como carteras.

No crear solicitudes ni reservas en Producción.

==================================================
6. GATES DE SEGURIDAD
==================================================

Durante toda la continuación:

PRODUCTION_READ_ONLY = true
POLICY_ACTIVATION_ALLOWED = false
ECONOMIC_EXECUTION_ALLOWED = false
PUBLIC_ENDPOINTS_ALLOWED = false
PRODUCTION_UI_CHANGES_ALLOWED = false

Todas las escrituras y pruebas:

LAB_ONLY

La migración v30:

- debe ser transaccional;
- idempotente;
- validada con integrity_check;
- validada con foreign_key_check;
- con rollback vacío probado;
- sin alterar las 82 solicitudes históricas.

==================================================
7. COMMIT
==================================================

No crear commit vacío.

Si se completa solo 2F-A1.1:

feat(lens): add atomic paper cash reservations

Si se completa todo 2F-A1:

feat(lens): add atomic paper allocation and exit reservations

Si ya existe un commit parcial:

- no crear commits duplicados;
- explicar si se conserva y se añade un segundo commit;
- o consolidar únicamente si todavía no se compartió y resulta seguro.

No hacer push.
No crear tag.

==================================================
8. ENTREGA DE RECUPERACIÓN
==================================================

RECOVERY_STATUS =
NO_CHANGES /
PARTIAL_UNCOMMITTED /
PARTIAL_COMMITTED /
IMPLEMENTATION_COMPLETE_TESTS_PENDING /
COMPLETE

CANONICAL_HEAD_ORIGINAL = 0466595
HEAD_FOUND_AT_RECOVERY = <hash>
CANONICAL_HEAD_AFTER = <hash/UNCHANGED>

PRODUCTION_SCHEMA = v29
LAB_SCHEMA = <v29/v30>

PRODUCTION_READ_ONLY = PASS/FAIL
PRODUCTION_DATA_UNCHANGED = PASS/FAIL
NO_POLICY_ACTIVATED = PASS/FAIL
NO_ECONOMIC_EXECUTION = PASS/FAIL

FILES_FOUND_MODIFIED = <lista>
FILES_FOUND_UNTRACKED = <lista>
COMMITS_FOUND = <lista>

LAST_COMPLETED_STEP = <paso>
FIRST_PENDING_STEP = <paso>
STOP_REASON = <causa>

V30_MIGRATION = NOT_STARTED/PARTIAL/PASS
REQUEST_LIFECYCLE = NOT_STARTED/PARTIAL/PASS
CASH_RESERVATIONS = NOT_STARTED/PARTIAL/PASS
CAPACITY_RACE_PROTECTION = NOT_STARTED/PARTIAL/PASS
POSITION_ACTION_LIFECYCLE = NOT_STARTED/PARTIAL/PASS
QUANTITY_RESERVATIONS = NOT_STARTED/PARTIAL/PASS
QUANTITY_RACE_PROTECTION = NOT_STARTED/PARTIAL/PASS
CATALYST_ALERT_RECONCILIATION = NOT_STARTED/PARTIAL/PASS

TESTS_PASS = <número>
TESTS_FAIL = <número>
TESTS_PENDING = <lista>

LOCAL_COMMIT = <hash/NONE>

EXACT_NEXT_STEP:
<una acción concreta>

Si todo 2F-A1 no cabe de forma segura, detenerse únicamente después de dejar
2F-A1.1 completo, probado y coherente.