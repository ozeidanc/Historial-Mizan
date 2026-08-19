CORRECCIÓN DE HECHO OPERATIVO · PANW

PANW fue vendida realmente en Wio ayer, 2026-07-27.

Ya no forma parte de la cartera real.

La afirmación anterior de que PANW seguía abierta en Wio era incorrecta:
describía únicamente el estado no reconciliado de Mizan.

No cancelar la venta.
No recomprar PANW.
No reabrir automáticamente la posición.
No crear una orden compensatoria.

1. Auditar la venta real usando los datos de Wio que proporcione Omar:

- cantidad ejecutada;
- precio;
- comisión;
- fecha y hora;
- referencia Wio;
- estado FILLED o PARTIALLY_FILLED.

2. Determinar a qué recomendación o decisión operativa correspondió realmente
la orden enviada a Wio.

No asumir que fue recommendation_id=168 solo porque era la línea ELIMINAR
visible.

3. Reconciliar el fill por la ruta canónica post-trade:

- crear el movimiento real de venta;
- crear el settlement correspondiente;
- registrar la comisión;
- reducir o cerrar el holding;
- actualizar cash y NAV;
- preservar rentabilidad y coste correctamente;
- enlazar el fill con su procedencia real.

4. La decisión DECLINED de recommendation_id=168 ya existe y no debe borrarse.

Si la venta real corresponde a esa recomendación, añadir una rectificación
append-only que explique:

- la ejecución en Wio ocurrió antes de la contención;
- DECLINED reflejaba información operativa incompleta;
- el fill real se reconcilia posteriormente;
- la venta fue originada por SEALED_TIEBREAK_NOT_APPLIED.

No reescribir la decisión ni la línea histórica.

5. Preservar la conclusión forense:

- con el motor legacy, PANW quedó fuera;
- con el desempate sellado, PANW habría permanecido seleccionada;
- la venta ya ejecutada no se revierte automáticamente;
- cualquier futura reincorporación debe surgir de una revisión válida y no
  de una compensación automática.

Entregar:

PANW_WIO_EXECUTION_DATE =
2026-07-27

PANW_WIO_EXECUTION_STATUS =
FILLED/PARTIALLY_FILLED

PANW_EXECUTED_QUANTITY =
<cantidad>

PANW_EXECUTED_PRICE =
<precio>

PANW_EXECUTION_COMMISSION =
<comisión>

PANW_WIO_REFERENCE =
<referencia>

PANW_RECONCILIATION_SOURCE =
<recommendation_id/OTHER>

PANW_SELL_MOVEMENT_ID =
<id>

PANW_EXECUTION_LINK_ID =
<id/NONE>

PANW_HOLDING_AFTER_RECONCILIATION =
0/<cantidad residual>

PANW_CASH_EVENT_CREATED =
PASS/FAIL

PANW_COMMISSION_RECORDED =
PASS/FAIL

PANW_NAV_RECONCILED =
PASS/FAIL

RECOMMENDATION_168_HISTORY_PRESERVED =
PASS/FAIL

AUTOMATIC_PANW_REPURCHASE =
NO

BROKER_ORDERS_SENT =
0

EXACT_REMAINING_BLOCKERS =
NONE/<lista>

No inventar los datos del fill. Esperar a que Omar proporcione los valores
exactos visibles en Wio.