MIZAN · HOTFIX CRÍTICO
REPARACIÓN GLOBAL DE EJECUCIONES WIO DE LA REVISIÓN
NO ES UN CASO PANW · AFECTA A MÚLTIPLES RECOMENDACIONES

CONTEXTO

Omar ejecutó en Wio múltiples recomendaciones de la revisión de posiciones
existentes.

Al introducir las ejecuciones reales, Mizan bloquea múltiples registros con:

CASH_OVERSPEND_BLOCKED
requiere <gross + commission>
disponible 0,00

Esto no es un problema específico de PANW.

Es un defecto sistémico del flujo que intenta registrar fills reales ya
ejecutados utilizando una validación de capacidad previa a operar.

Aparcar completamente Gate 2F.

PRIORIDAD =
CRÍTICA

==================================================
1. OBJETIVO
==================================================

Reparar globalmente el flujo para que todas las ejecuciones reales de Wio:

- se registren con cantidad real;
- se registren con precio real;
- conserven el importe bruto real;
- registren la comisión separadamente;
- apliquen correctamente su efecto total sobre cash;
- creen el movimiento del Libro Mayor;
- actualicen el holding;
- queden vinculadas a su recomendación;
- no se dupliquen;
- no sean bloqueadas por una comprobación pre-trade;
- permitan reconciliar el lote completo.

No crear excepciones por ticker.

No hardcodear PANW.

La corrección debe aplicar a:

- INCORPORAR;
- AUMENTAR;
- REDUCIR;
- ELIMINAR;

y a cualquier otra acción que represente un fill real confirmado.

==================================================
2. RESTRICCIÓN OPERATIVA INMEDIATA
==================================================

No enviar ninguna orden a Wio.

No reejecutar operaciones.

No asumir que una recomendación pendiente significa que la operación no
ocurrió en el broker.

No borrar decisiones ni formularios conservados.

No modificar valores reales introducidos.

No sustituir datos reales por la recomendación.

No crear movimientos duplicados.

No inventar aportaciones para cuadrar cash.

No cambiar Book, holdings o cash hasta completar:

- backup;
- auditoría;
- preview;
- detección de duplicados.

==================================================
3. IDENTIFICAR LA REVISIÓN COMPLETA
==================================================

Localizar el review_run que contiene las recomendaciones mostradas.

La tabla visible contiene al menos 25 recomendaciones, pero no asumir que el
lote se limita a 25.

PANW aparece en un formulario de INCORPORAR y puede pertenecer:

- a la misma revisión;
- a otra sección del mismo run;
- a una recomendación relacionada.

Resolver mediante IDs y lineage.

Inventariar todas las recomendaciones del run:

- review_run_id;
- recommendation_id;
- ticker;
- action;
- recommended quantity;
- reference price;
- estimated amount;
- decision status;
- accepted_at;
- saved execution fields;
- existing movement_id;
- holding mutation;
- cash mutation;
- error registrado;
- idempotency key.

==================================================
4. CLASIFICACIÓN POR RECOMENDACIÓN
==================================================

Clasificar cada recomendación:

PENDING_NO_EXECUTION_DATA

- pendiente;
- sin datos de ejecución conservados;
- sin movimiento.

PENDING_WITH_EXECUTION_DATA

- pendiente o bloqueada;
- existen cantidad/precio/comisión/fecha;
- sin movimiento.

ACCEPTED_AND_RECORDED

- decisión aceptada;
- movimiento existente y vinculado;
- holding actualizado;
- cash contabilizado.

ACCEPTED_ORPHANED

- aparece aceptada;
- no existe movimiento vinculado.

MOVEMENT_WITHOUT_ACCEPTANCE

- movimiento existente;
- decisión no refleja el registro.

PARTIAL_WRITE

Existe solo parte de:

- movimiento;
- execution detail;
- holding;
- cash;
- decisión;
- lineage.

DUPLICATE_RISK

- existen varios movimientos candidatos;
- o el fill podría haberse registrado con otra referencia.

MANUAL_EXECUTION_DATA_REQUIRED

- Omar confirma que se ejecutó;
- faltan uno o más datos reales necesarios.

Entregar el inventario completo antes de reparar.

==================================================
5. RECUPERAR LOS FORMULARIOS CONSERVADOS
==================================================

El sistema afirmó:

“datos conservados, puedes reintentar”.

Localizar esos datos en:

- base de datos;
- audit events;
- session storage;
- local storage;
- estado del frontend;
- logs de requests;
- payloads de error;
- drafts.

Exportarlos antes de reiniciar o desplegar.

Por formulario recuperar:

- recommendation_id;
- ticker;
- action;
- executed quantity;
- executed price;
- commission;
- execution date;
- optional execution time;
- input mode;
- error;
- saved_at;
- payload hash.

Declarar:

SAVED_EXECUTION_FORMS_FOUND
SAVED_EXECUTION_FORMS_COMPLETE
SAVED_EXECUTION_FORMS_INCOMPLETE
SAVED_EXECUTION_FORMS_UNLINKED

No usar los valores recomendados para rellenar campos reales ausentes.

==================================================
6. SEMÁNTICA CONTABLE GLOBAL
==================================================

Para toda COMPRA:

executed_gross_amount =
executed_quantity × executed_price

commission_amount =
commission

cash_effect =
-(executed_gross_amount + commission_amount)

Para toda VENTA:

executed_gross_amount =
executed_quantity × executed_price

commission_amount =
commission

cash_effect =
executed_gross_amount - commission_amount

Persistir separadamente:

- recommended_quantity;
- reference_price;
- estimated_recommendation_amount;
- executed_quantity;
- executed_price;
- executed_gross_amount;
- commission_amount;
- executed_cash_effect;
- execution_date;
- execution_time nullable;
- execution_source = WIO_MANUAL_CONFIRMATION;
- slippage_amount;
- slippage_percentage.

Nunca guardar gross + commission como importe comprado.

Nunca guardar gross - commission como importe vendido.

El principal del movimiento es el gross.

La comisión es independiente.

El cash effect es la suma algebraica final.

==================================================
7. EJEMPLO PANW COMO TEST, NO COMO EXCEPCIÓN
==================================================

PANW:

executed_quantity =
0,33422822

executed_price =
331,51

executed_gross_amount =
110,7999972122

executed_gross_amount_display =
110,80

commission =
0,52

executed_cash_effect =
-111,3199972122

total_cash_debit_display =
111,32

La recomendación:

0,342213 × 323,92 ≈ 110,85

es solo una estimación.

Resultado correcto:

PANW_LEDGER_PRINCIPAL =
110,80

PANW_LEDGER_COMMISSION =
0,52

PANW_CASH_DEBIT =
111,32

Pero la misma regla debe aplicarse a todas las recomendaciones del lote.

==================================================
8. DOS MODOS OPERATIVOS DIFERENTES
==================================================

Implementar explícitamente:

A. PRE_TRADE_PROPOSAL

La operación no ha ocurrido.

En este modo:

- validar cash;
- permitir CASH_OVERSPEND_BLOCKED;
- no crear movimiento si no hay capacidad.

B. BROKER_EXECUTED_FILL

La operación ya ocurrió en Wio.

En este modo:

- registrar la ejecución real;
- no aplicar CASH_OVERSPEND_BLOCKED;
- actualizar Libro Mayor;
- actualizar holding;
- contabilizar cash;
- registrar comisión;
- conservar lineage;
- marcar reconciliación pendiente si no cuadra el cash.

El bypass del bloqueo de cash solo puede utilizarse cuando exista:

- confirmación explícita de fill ejecutado;
- cantidad real;
- precio real;
- comisión;
- fecha;
- source = WIO_MANUAL_CONFIRMATION;
- idempotency key.

No desactivar globalmente el control pre-trade.

==================================================
9. ROOT CAUSE OBLIGATORIO
==================================================

Trazar el flujo completo y determinar:

ROOT_CAUSE_CASH_ZERO

Comprobar si:

- las ventas no se registraron;
- sus proceeds no se contabilizaron;
- se usa una fuente incorrecta de cash;
- existe fallback a cero;
- la revisión no procesa el lote;
- la lectura ocurre antes de COMMIT;
- las comisiones provocan sobregiro;
- el cash inicial no estaba reconciliado.

ROOT_CAUSE_LEDGER_NOT_WRITTEN

Comprobar si:

- el bloqueo ocurre antes del INSERT;
- la decisión se guarda en otra transacción;
- el movimiento falla después de aceptar;
- existe rollback parcial;
- se escribe en una DB distinta;
- el Book lee otra fuente;
- falta linkage recommendation→movement.

ROOT_CAUSE_FALSE_BOOK_SUCCESS

Comprobar si la UI muestra:

“Aceptada / Registrada en el Book”

basándose únicamente en:

decision.status = ACCEPTED

en lugar de verificar:

movement_id committed.

No desplegar el parche sin identificar estas causas.

==================================================
10. ATOMICIDAD POR FILL
==================================================

Cada fill debe registrarse en una transacción SQLite:

1. cargar recommendation;
2. comprobar idempotencia;
3. comprobar duplicados económicos;
4. validar execution fields;
5. calcular gross;
6. calcular commission;
7. calcular cash effect;
8. insertar execution detail;
9. insertar movimiento del Libro Mayor;
10. aplicar cash effect;
11. actualizar o crear holding;
12. vincular recommendation y movement;
13. marcar decisión aceptada;
14. registrar audit event;
15. COMMIT.

Si falla:

- ROLLBACK completo;
- decisión no aceptada;
- cero movimiento parcial;
- cero cambio de holding;
- cero cambio de cash;
- formulario conservado.

Invariante:

DECISION_ACCEPTED
SI Y SOLO SI
LEDGER_MOVEMENT_COMMITTED_AND_LINKED

==================================================
11. RECONCILIACIÓN DEL LOTE
==================================================

Además de la atomicidad por fill, reconciliar todo el batch.

Calcular:

opening_book_cash

total_sell_gross

total_sell_commissions

net_sell_proceeds =
total_sell_gross - total_sell_commissions

total_buy_gross

total_buy_commissions

total_buy_debits =
total_buy_gross + total_buy_commissions

net_batch_cash_effect =
net_sell_proceeds - total_buy_debits

closing_calculated_book_cash =
opening_book_cash + net_batch_cash_effect

No bloquear un fill ya ejecutado debido al orden en que Omar abre los
formularios.

No asumir que la secuencia de aceptación en Mizan coincide con la secuencia
de ejecución en Wio.

Si existen timestamps reales:

- ordenar por timestamp.

Si solo existe fecha:

- agrupar por execution_batch_id;
- no inventar horas;
- registrar ORDER_WITHIN_DAY_UNKNOWN.

==================================================
12. CASH NEGATIVO O NO RECONCILIADO
==================================================

Una ejecución real debe registrarse aunque el cash interno previo no baste.

No inventar una aportación.

No cambiar el fill.

Después de contabilizar el lote:

Si closing book cash no reconcilia:

CASH_RECONCILIATION_REQUIRED

Registrar:

- diferencia;
- operaciones incluidas;
- cash inicial;
- ventas;
- compras;
- comisiones;
- cash calculado;
- cash del broker, si se proporciona.

Si no se conoce el cash final de Wio:

BROKER_CASH_NOT_PROVIDED

No declarar:

CASH_RECONCILED

sin evidencia.

==================================================
13. REGISTRO MASIVO CONTROLADO
==================================================

Crear una herramienta:

previewBrokerExecutedFillBatch

Debe producir una tabla por ticker:

- recommendation ID;
- action;
- recommended quantity;
- estimated amount;
- executed quantity;
- executed price;
- gross amount;
- commission;
- cash effect;
- date/time;
- current status;
- existing movement;
- duplicate risk;
- proposed repair;
- missing data.

No escribir.

Después crear:

recordBrokerExecutedFillBatch

Solo debe procesar fills:

- completos;
- confirmados;
- sin ambigüedad;
- sin movimiento equivalente.

Para datos incompletos:

- no inventar;
- dejar MANUAL_EXECUTION_DATA_REQUIRED.

La herramienta debe poder:

- ejecutar todo el batch en una transacción global;
- o ejecutar fills atómicos con un batch reconciliation record.

Preferencia:

fills atómicos + batch reconciliation durable.

Así, un ticker incompleto no impide conservar correctamente los demás.

==================================================
14. REPARACIÓN DE ACEPTACIONES HUÉRFANAS
==================================================

Crear:

repairAcceptedRecommendationsWithoutMovement

Para cada aceptación huérfana:

- recuperar fill real;
- buscar movimientos equivalentes;
- verificar duplicados;
- crear execution y movement;
- aplicar holding y cash;
- vincular movement_id;
- registrar repair event.

No aceptar silenciosamente una recomendación sin fill real.

Si faltan datos:

MANUAL_EXECUTION_DATA_REQUIRED

Segunda ejecución:

ALREADY_REPAIRED

No duplicar.

==================================================
15. UI CORREGIDA
==================================================

El formulario debe preguntar claramente:

ESTADO DE LA OPERACIÓN

- Todavía no ejecutada;
- Ya ejecutada en Wio.

Si:

YA EJECUTADA EN WIO

mostrar campos:

- cantidad realmente ejecutada;
- precio real;
- comisión;
- fecha;
- hora opcional;
- importe bruto calculado;
- débito/abono total calculado.

Permitir dos modos de entrada:

A. CANTIDAD + PRECIO

B. IMPORTE BRUTO + PRECIO

En modo B:

derived_quantity =
gross_amount / executed_price

La comisión siempre es separada.

No confundir:

- importe bruto;
- importe estimado;
- total de cash.

Antes de confirmar mostrar:

- recomendación original;
- ejecución real;
- diferencia;
- comisión;
- cash effect.

==================================================
16. ESTADO DE ÉXITO
==================================================

Mostrar:

“Aceptada · Registrada en el Libro Mayor”

solo si:

- movement_id existe;
- transaction committed;
- holding actualizado;
- cash effect contabilizado;
- recommendation vinculada.

Si se registró pero cash necesita conciliación:

“Aceptada · Registrada · Conciliación de cash pendiente”

Mostrar movement_id.

No mostrar éxito basándose solo en la decisión.

==================================================
17. IDEMPOTENCIA Y DUPLICADOS
==================================================

Construir idempotency key a partir de identidad durable, no solo ticker:

- recommendation_id;
- execution date;
- action;
- executed quantity;
- executed price;
- commission;
- optional broker execution reference.

Antes de insertar buscar coincidencias por:

- ticker;
- side;
- date;
- quantity;
- price;
- gross;
- commission;
- recommendation lineage.

Mismo fill repetido:

ALREADY_RECORDED

Mismo recommendation con datos diferentes:

EXECUTION_DATA_CONFLICT

No crear dos movimientos.

==================================================
18. PRUEBAS
==================================================

Crear:

verify-broker-executed-fill-mode.mjs
verify-broker-fill-gross-amount.mjs
verify-broker-fill-commission-separation.mjs
verify-broker-fill-cash-effect.mjs
verify-broker-fill-pretrade-guard.mjs
verify-broker-fill-ledger-atomicity.mjs
verify-broker-fill-batch-reconciliation.mjs
verify-broker-fill-orphan-repair.mjs
verify-broker-fill-idempotency.mjs
verify-broker-fill-ui-status.mjs
verify-broker-fill-form-recovery.mjs

Casos obligatorios:

A. PANW gross = 110,80.
B. PANW commission = 0,52.
C. PANW debit = 111,32.
D. Diferencia con 110,85 no bloquea.
E. Otra compra afectada usa la misma regla.
F. Venta gross y proceeds netos separados.
G. Pre-trade sin cash sigue bloqueando.
H. Broker fill sin cash se registra con reconciliation warning.
I. Venta del lote incrementa cash.
J. Compra del lote reduce cash.
K. Orden de apertura de formularios no bloquea fills.
L. Aceptación sin movimiento es imposible.
M. Reparación de huérfano no duplica.
N. Formulario se conserva tras error.
O. UI muestra movement_id.
P. Holdings usan cantidades reales.
Q. Comisiones quedan separadas.
R. Batch produce resumen de conciliación.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

==================================================
19. PRODUCCIÓN
==================================================

Antes:

- detener escrituras;
- backup;
- SHA-256;
- integrity_check;
- foreign_key_check;
- exportar formularios;
- capturar baseline;
- generar preview de batch;
- revisar duplicados;
- ejecutar tests.

Orden de despliegue:

1. corregir atomicidad;
2. corregir modo BROKER_EXECUTED_FILL;
3. corregir estado UI;
4. reiniciar una sola instancia;
5. verificar health;
6. generar preview;
7. registrar fills completos;
8. reparar aceptaciones huérfanas;
9. reconciliar batch;
10. verificar Libro Mayor;
11. verificar holdings;
12. verificar cash;
13. recalcular NAV.

No enviar órdenes a Wio.

==================================================
20. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = <hash>
CANONICAL_HEAD_AFTER = <hash>

ROOT_CAUSE_CASH_ZERO = <causa>
ROOT_CAUSE_LEDGER_NOT_WRITTEN = <causa>
ROOT_CAUSE_FALSE_BOOK_SUCCESS = <causa>

REVIEW_RUN_ID = <id>
RECOMMENDATIONS_IN_BATCH = <número>

SAVED_EXECUTION_FORMS_FOUND = <número>
SAVED_EXECUTION_FORMS_COMPLETE = <número>
SAVED_EXECUTION_FORMS_INCOMPLETE = <número>

PENDING_WITH_EXECUTION_DATA_BEFORE = <número>
ACCEPTED_AND_RECORDED_BEFORE = <número>
ACCEPTED_ORPHANED_BEFORE = <número>
PARTIAL_WRITES_BEFORE = <número>
DUPLICATE_RISKS = <número>

BROKER_EXECUTED_FILL_MODE = PASS/FAIL
PRE_TRADE_CASH_GUARD_PRESERVED = PASS/FAIL
COMMISSION_SEPARATED_FROM_PRINCIPAL = PASS/FAIL

FILLS_RECORDED = <número>
ORPHANED_ACCEPTANCES_REPAIRED = <número>
MANUAL_EXECUTION_DATA_REQUIRED = <número>
DUPLICATES_CREATED = 0

TOTAL_BUY_GROSS = <importe>
TOTAL_BUY_COMMISSIONS = <importe>
TOTAL_BUY_DEBITS = <importe>

TOTAL_SELL_GROSS = <importe>
TOTAL_SELL_COMMISSIONS = <importe>
NET_SELL_PROCEEDS = <importe>

NET_BATCH_CASH_EFFECT = <importe>
OPENING_BOOK_CASH = <importe>
CLOSING_CALCULATED_BOOK_CASH = <importe>
BROKER_CASH = <importe/NOT_PROVIDED>
CASH_RECONCILIATION_DIFFERENCE = <importe/UNKNOWN>
CASH_RECONCILIATION_STATUS =
PASS /
REQUIRED /
BROKER_CASH_NOT_PROVIDED

DECISIONS_WITHOUT_MOVEMENT_AFTER = 0
PARTIAL_WRITES_AFTER = 0
DUPLICATE_MOVEMENTS_AFTER = 0

HOLDINGS_RECONCILED = PASS/FAIL
BOOK_RECONCILED = PASS/FAIL
LEDGER_RECONCILED = PASS/FAIL
CASH_RECONCILED = PASS/FAIL/NOT_VERIFIABLE
NAV_RECALCULATED = PASS/FAIL

PANW_GROSS = 110,80
PANW_COMMISSION = 0,52
PANW_CASH_DEBIT = 111,32
PANW_REGISTERED = PASS/FAIL

NO_WIO_ORDERS_SENT = PASS/FAIL
NO_DUPLICATE_EXECUTIONS = PASS/FAIL
NO_PUSH_PER
