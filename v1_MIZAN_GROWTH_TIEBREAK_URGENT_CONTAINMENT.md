MIZAN · CONTENCIÓN URGENTE PANW
Y CORRECCIÓN VERSIONADA DEL DESEMPATE SELLADO

El archivo normativo:

spec-mizan-c2-seleccion.md

precede íntegramente a esta orden.

La auditoría queda aceptada.

DECISIONES

PANW_RECOMMENDATION_168_EXECUTION =
NOT_AUTHORIZED

TIEBREAK_FIX_ROUTE =
A_VERSIONED_ENGINE

PRODUCTIVE_FUNDAMENTAL_CACHE_REFRESH =
NOT_AUTHORIZED

C2_IS_NEW_METHODOLOGY =
YES

==================================================
1. CONTENCIÓN URGENTE DE PANW
==================================================

No ejecutar:

recommendation_id =
168

ticker =
PANW

action =
ELIMINAR

La evidencia confirma:

- PANW fue comprada mediante recommendation_id 130;
- movement_id 91 registró la compra;
- la posición permanece abierta;
- recommendation_id 168 no se ha ejecutado;
- no existe movimiento de venta;
- la pérdida aproximada es latente, no realizada;
- con el desempate sellado PANW queda seleccionado en rank 25;
- la recomendación ELIMINAR es consecuencia directa del defecto de
  implementación.

Impedir que recommendation_id 168 pueda:

- aceptarse;
- generar formulario Wio;
- registrarse como ejecución;
- crear movimiento;
- reducir holdings;
- generar settlement.

Preservar la fila y toda su auditoría.

No borrar ni reescribir historia.

Utilizar un estado canónico existente equivalente a:

INVALIDATED_BY_SELECTION_ENGINE_DEFECT

o, si no existe, añadir un estado operativo compatible y auditado.

Persistir como razón:

SEALED_TIEBREAK_NOT_APPLIED

La invalidación no debe interpretarse como:

- DECLINED por decisión de inversión;
- venta ejecutada;
- recomendación superseded por cambio económico;
- modificación de la tesis de PANW.

Entregar:

PANW_168_STATUS_BEFORE =
<estado>

PANW_168_STATUS_AFTER =
<estado>

PANW_168_EXECUTABLE_AFTER =
NO

PANW_POSITION_QUANTITY_UNCHANGED =
PASS

PANW_SELL_MOVEMENTS_CREATED =
0

PANW_CASH_EVENTS_CREATED =
0

BROKER_ORDERS_SENT =
0

==================================================
2. ALCANCE DE LA REVISIÓN AFECTADA
==================================================

Auditar la review session:

session_id =
8

Determinar qué líneas dependen del desempate defectuoso.

No asumir que PANW es la única afectada.

Comparar para la sesión:

LEGACY_ENGINE_SELECTION

contra:

SEALED_TIEBREAK_COMPLIANT_SELECTION

Entregar:

- top 25 legacy;
- top 25 conforme;
- diferencias;
- recomendaciones afectadas;
- decisiones ya tomadas;
- ejecuciones ya registradas;
- recomendaciones todavía pendientes.

Toda recomendación pendiente cuya existencia dependa del defecto debe quedar
no accionable hasta completar el preview.

No revertir:

- operaciones ya ejecutadas;
- movimientos ya registrados;
- holdings existentes;
- settlements;
- comisiones.

Entregar:

SESSION_8_PENDING_LINES_AFFECTED =
<lista>

SESSION_8_EXECUTED_LINES_AFFECTED =
<lista>

SESSION_8_INVALIDATIONS_REQUIRED =
<lista>

==================================================
3. RAMA SEPARADA
==================================================

No implementar este hotfix en:

feature/c2-continuous-scoring

Crear desde el HEAD productivo/local aplicable:

hotfix/growth-sealed-tiebreak-compliance

Antes:

- verificar el grafo;
- inventariar commits no desplegados;
- no perder 23978af;
- no mezclar cambios de Track record;
- no mezclar Cockpit V2;
- no mezclar el score continuo C2.

==================================================
4. MOTOR VERSIONADO
==================================================

Conservar dos implementaciones explícitas:

LEGACY

Orden:

score DESC
→ greens DESC
→ ticker ASC

Uso:

- reproducción histórica;
- sesiones anteriores al effective_from;
- test dorado.

SEALED_TIEBREAK_COMPLIANT

Orden:

score DESC
→ greens DESC
→ revGrowthPct DESC
→ ticker ASC

Uso:

- sesiones futuras desde effective_from;
- cumplimiento del contrato sellado.

Añadir:

selection_engine_version

Valores mínimos:

legacy
sealed-tiebreak-compliant

La versión debe quedar registrada por sesión y en el audit trail.

No inferir la versión posteriormente mediante la fecha si puede persistirse
explícitamente.

No modificar:

- crecimiento-v2-c0-sealed.mjs;
- methodology_hash de C0;
- archivo sellado C1;
- methodology_hash compartido;
- score;
- gates;
- topN;
- weights;
- sizing;
- materialidad de 2 USD.

==================================================
5. REPLAY HISTÓRICO
==================================================

Las ocho sesiones históricas deben reproducirse con:

selection_engine_version =
legacy

Resultado obligatorio:

- selección byte-idéntica;
- orden byte-idéntico;
- selection_content_hash idéntico;
- methodology_hash idéntico;
- audit trail intacto.

No recalcular las sesiones históricas con el motor conforme.

Entregar:

LEGACY_GOLDEN_SESSIONS =
8

LEGACY_REPLAY_BYTE_IDENTICAL =
PASS/FAIL

C0_METHODOLOGY_HASH_UNCHANGED =
PASS/FAIL

C1_METHODOLOGY_HASH_UNCHANGED =
PASS/FAIL

==================================================
6. PREVIEW FORWARD DEL MOTOR CONFORME
==================================================

Antes de activarlo, ejecutar un preview read-only usando exactamente los
inputs actualmente observados.

Mostrar:

- top 25 legacy;
- top 25 conforme;
- altas;
- bajas;
- ranks;
- score;
- greens;
- total;
- revGrowthPct;
- posiciones actuales;
- recomendaciones que aparecerían;
- importe bruto;
- materialidad;
- cash;
- holdings;
- impacto potencial.

Prestar atención a:

- ABNB;
- AMD;
- CDNS;
- PANW;
- TTWO;
- WDAY.

No crear recomendaciones productivas en el preview.

No vender ABNB automáticamente.

No comprar TTWO, PANW, WDAY ni ningún otro ticker automáticamente.

Entregar:

FORWARD_LEGACY_SELECTION =
<lista>

FORWARD_COMPLIANT_SELECTION =
<lista>

FORWARD_ADDITIONS =
<lista>

FORWARD_REMOVALS =
<lista>

PENDING_RECOMMENDATIONS_INVALIDATED =
<lista>

EXECUTED_RECOMMENDATIONS_MODIFIED =
0

==================================================
7. ACTIVACIÓN SEPARADA
==================================================

Implementar el motor conforme, pero dejar su activación pendiente.

Estado inicial:

SEALED_TIEBREAK_COMPLIANT_ENGINE =
READY_NOT_ACTIVE

effective_from =
NULL

La activación debe ser una operación independiente, con:

- effective_from futuro;
- sesión bursátil segura;
- preview aprobado;
- cero fills pendientes;
- selección activa sellada;
- lineage;
- rollback a legacy en una operación.

No cambiar la selección activa durante el despliegue del código.

No generar automáticamente una revisión por desplegar.

==================================================
8. SNAPSHOT AS-OBSERVED
==================================================

En paralelo, preparar el archivo:

AS_OBSERVED_INPUT_SNAPSHOT

Debe capturar lo que C0/C1 consume antes de cualquier refresh:

- ticker;
- factor;
- raw value;
- binary state;
- cache mtime;
- fetchedAt;
- acceptedDate;
- period end;
- price as-of;
- source;
- missing state;
- input hash;
- content hash.

Este snapshot:

- es append-only;
- es idempotente;
- no sustituye C0_SELECTION_FROZEN;
- no modifica C0/C1;
- no cambia la selección;
- no se presenta como histórico PIT retroactivo.

No activar todavía el refresh de fundamentales.

==================================================
9. CACHÉ DE FUNDAMENTALES
==================================================

Registrar como defecto separado:

CACHE_REFRESH_POLICY =
WRITE_ONCE

No corregirlo en este hotfix.

No sustituir:

backtest/.cache/fund_<ticker>.json

No ejecutar:

refresh-fundamentals.mjs

contra la caché productiva.

El rediseño futuro deberá separar:

- fetch;
- staging;
- validación;
- snapshot PIT;
- promoción de datos;
- selección;
- activación de composición.

==================================================
10. TESTS
==================================================

Probar:

A. legacy reproduce las ocho sesiones;
B. compliant aplica revGrowthPct_desc;
C. ticker solo decide empates residuales;
D. PANW queda rank 25 en la sesión 07-27;
E. recommendation 168 no es ejecutable;
F. no se crea venta de PANW;
G. no cambia ningún holding;
H. no se crea cash event;
I. no cambia methodology_hash;
J. despliegue no cambia active selection;
K. preview no escribe;
L. rollback selecciona legacy;
M. C2 permanece inactivo;
N. cero órdenes Wio.

==================================================
11. COMMITS
==================================================

Crear commits separados:

1.
docs(growth): add normative C2 selection specification

Este commit puede permanecer en la rama C2 si corresponde.

En la rama del hotfix:

2.
fix(growth): preserve invalid PANW exit recommendation

3.
fix(growth): version sealed selection tie-break implementation

4.
test(growth): preserve legacy selection replay

No hacer push.
No crear tag.

==================================================
12. CHECKPOINT ANTES DE DESPLIEGUE
==================================================

Entregar:

NORMATIVE_SPEC_PRESENT =
PASS/FAIL

PANW_RECOMMENDATION_168_CONTAINED =
PASS/FAIL

PANW_RECOMMENDATION_168_EXECUTABLE =
NO

SESSION_8_AFFECTED_LINES_AUDITED =
PASS/FAIL

HOTFIX_BRANCH =
hotfix/growth-sealed-tiebreak-compliance

LEGACY_ENGINE_AVAILABLE =
PASS/FAIL

COMPLIANT_ENGINE_AVAILABLE =
PASS/FAIL

COMPLIANT_ENGINE_STATUS =
READY_NOT_ACTIVE

COMPLIANT_ENGINE_EFFECTIVE_FROM =
NULL

LEGACY_REPLAY_BYTE_IDENTICAL =
PASS/FAIL

C0_HASH_UNCHANGED =
PASS/FAIL

C1_HASH_UNCHANGED =
PASS/FAIL

FORWARD_PREVIEW =
PASS/FAIL

ACTIVE_SELECTION_CHANGED =
NO

HISTORICAL_RECOMMENDATIONS_REWRITTEN =
0

EXISTING_MOVEMENTS_MODIFIED =
0

EXISTING_HOLDINGS_MODIFIED =
0

PRODUCTION_BROKER_ORDERS_SENT =
0

AS_OBSERVED_SNAPSHOT_READY =
PASS/FAIL

PRODUCTIVE_CACHE_REFRESH_PERFORMED =
NO

EXACT_DECISIONS_REQUIRED_FROM_OMAR =
<lista/NONE>

Detenerse antes del despliegue y de activar el motor conforme.

No comenzar el score continuo C2.