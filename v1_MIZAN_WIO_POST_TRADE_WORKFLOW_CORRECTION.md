CORRECCIÓN DE CONTRATO FUNCIONAL

Todas las recomendaciones incluidas en este flujo se ejecutan primero
manualmente en Wio.

Después, Omar pulsa Aceptar e introduce en Mizan los datos exactos de la
operación ya ejecutada.

Por tanto, el flujo real es siempre:

RECOMENDACIÓN
→ EJECUCIÓN MANUAL EN WIO
→ INTRODUCCIÓN DE DATOS REALES EN MIZAN
→ REGISTRO EN LIBRO MAYOR
→ ACTUALIZACIÓN DE HOLDING Y CASH

No existe dentro del formulario de aceptación el modo:

PRE_TRADE_PROPOSAL

No preguntar:

- “Todavía no ejecutada”;
- “Ya ejecutada en Wio”.

Toda aceptación de una recomendación en este formulario significa:

BROKER_EXECUTED_FILL

==================================================
1. SEMÁNTICA DE ACEPTAR
==================================================

El botón actual:

ACEPTAR

debe entenderse y, preferiblemente, etiquetarse como:

REGISTRAR EJECUCIÓN WIO

o:

ACEPTAR Y REGISTRAR EJECUCIÓN REAL

Al abrir el formulario:

- la operación ya ocurrió;
- Mizan no autoriza la operación;
- Mizan no decide si existe cash suficiente;
- Mizan registra la ejecución confirmada por Omar;
- los campos editables representan la fuente de verdad contable.

Los valores de la recomendación son exclusivamente referencia:

- cantidad recomendada;
- precio EOD;
- importe estimado.

No son límites de la ejecución real.

==================================================
2. ELIMINAR CASH_OVERSPEND_BLOCKED DEL REGISTRO
==================================================

La ruta que registra una recomendación aceptada no puede lanzar:

CASH_OVERSPEND_BLOCKED

porque la operación ya se ejecutó en Wio.

Eliminar esa validación del camino de registro post-trade.

No eliminarla de otros flujos futuros que realmente intenten autorizar una
operación antes de ejecutarla.

En este formulario, si:

required_cash > internal_available_cash

hacer:

1. registrar el fill real;
2. registrar el movimiento;
3. actualizar el holding;
4. aplicar el efecto de cash;
5. vincular la recomendación;
6. marcar la decisión como aceptada;
7. mostrar una advertencia de conciliación.

Estado:

EXECUTION_RECORDED_RECONCILIATION_REQUIRED

No:

CASH_OVERSPEND_BLOCKED

==================================================
3. TODAS LAS RECOMENDACIONES ESTÁN EJECUTADAS
==================================================

Omar confirma que todas las recomendaciones del lote fueron ejecutadas
previamente en Wio.

Por tanto, una recomendación todavía pendiente en Mizan significa:

EJECUTADA_EN_WIO_PENDIENTE_DE_REGISTRO_EN_MIZAN

No significa:

OPERACIÓN_NO_EJECUTADA

Clasificar el lote considerando esta confirmación del propietario.

Para cada recomendación debe ocurrir una de estas situaciones:

A. EXECUTION_DATA_COMPLETE

- se conservan cantidad real;
- precio real;
- comisión;
- fecha;
- puede registrarse.

B. EXECUTION_DATA_REQUIRED

- la operación ocurrió en Wio;
- faltan datos reales en Mizan;
- solicitar a Omar los datos exactos;
- no utilizar los valores recomendados como sustituto.

C. ALREADY_RECORDED

- ya existe movimiento correcto y vinculado;
- no duplicar.

D. ACCEPTED_ORPHANED

- figura aceptada;
- no existe movimiento;
- reparar utilizando los datos reales conservados.

E. PARTIAL_WRITE

- existe una parte del registro;
- reparar de forma idempotente.

No clasificar ninguna recomendación de este lote como:

PENDING_NO_EXECUTION

==================================================
4. CAMPOS AUTORITATIVOS
==================================================

Para cada operación, Omar introduce:

- cantidad realmente ejecutada;
- precio real de ejecución;
- comisión;
- fecha de ejecución;
- hora opcional;
- referencia del broker opcional.

Estos son los valores autoritativos.

Para una compra:

gross_amount =
executed_quantity × executed_price

cash_effect =
-(gross_amount + commission)

Para una venta:

gross_amount =
executed_quantity × executed_price

cash_effect =
gross_amount - commission

Persistir separadamente:

- executed_quantity;
- executed_price;
- gross_amount;
- commission;
- cash_effect.

La recomendación conserva separadamente:

- recommended_quantity;
- reference_price;
- estimated_amount.

No sustituir unos por otros.

==================================================
5. EL CASH INTERNO NO AUTORIZA EL FILL
==================================================

El cash de Mizan es un resultado contable que debe reconciliarse con lo que
ocurrió en Wio.

No es una autorización para registrar o rechazar una ejecución pasada.

Si Mizan muestra:

available_cash = 0

pero Wio ejecutó la compra:

- registrar la compra;
- no modificar sus datos;
- no inventar una aportación;
- no rechazarla;
- marcar la diferencia para conciliación.

La causa de available_cash = 0 debe investigarse por separado:

- ventas del lote todavía no registradas;
- proceeds ausentes;
- orden de aceptación distinto del orden de ejecución;
- cash inicial incorrecto;
- fuente de cash incorrecta;
- movimientos huérfanos;
- comisiones;
- otra diferencia contable.

==================================================
6. ORDEN DE REGISTRO EN MIZAN
==================================================

El orden en que Omar acepta las recomendaciones en Mizan no tiene por qué
coincidir con el orden de ejecución en Wio.

Por tanto, Mizan no puede bloquear una compra porque sus ventas relacionadas
todavía no hayan sido introducidas en la interfaz.

Registrar cada fill real de forma atómica.

Después reconciliar el lote completo mediante:

opening_book_cash
+ net_sell_proceeds
- total_buy_debits
=
closing_calculated_cash

Si existe hora real de ejecución:

- utilizarla.

Si solo existe fecha:

- no inventar hora;
- agrupar mediante execution_batch_id;
- marcar ORDER_WITHIN_DAY_UNKNOWN.

==================================================
7. PANW
==================================================

PANW debe registrarse exactamente así:

executed_quantity =
0.33422822

executed_price =
331.51

gross_amount =
110.7999972122

gross_amount_display =
110.80

commission =
0.52

cash_effect =
-111.3199972122

cash_debit_display =
111.32

La operación comprada fue:

110.80 USD

La comisión fue:

0.52 USD

El débito total de cash fue:

111.32 USD

La estimación original de 110.85 USD no debe bloquear ni sustituir la
ejecución.

==================================================
8. UI
==================================================

Eliminar del formulario cualquier selector entre:

- operación pendiente;
- operación ejecutada.

El formulario es exclusivamente de registro post-trade.

Mostrar:

EJECUCIÓN REAL EN WIO

Campos:

- cantidad ejecutada;
- precio ejecutado;
- importe bruto calculado;
- comisión;
- débito o abono neto;
- fecha;
- hora opcional;
- referencia Wio opcional.

Mostrar también, de forma secundaria:

RECOMENDACIÓN ORIGINAL

- cantidad recomendada;
- precio de referencia;
- importe estimado.

Botón:

REGISTRAR EJECUCIÓN WIO

No mostrar:

CASH_OVERSPEND_BLOCKED

En caso de diferencia de cash:

“Ejecución registrada. La conciliación de cash está pendiente.”

==================================================
9. ATOMICIDAD
==================================================

El registro de cada fill debe crear atómicamente:

- execution detail;
- movimiento del Libro Mayor;
- efecto de cash;
- actualización del holding;
- vínculo con recommendation;
- decisión aceptada;
- audit event.

Solo después del COMMIT mostrar:

“Aceptada · Registrada en el Libro Mayor”

Si falla:

- rollback completo;
- decisión pendiente de registro;
- datos del formulario conservados;
- no mostrar éxito.

==================================================
10. REPARACIÓN DEL LOTE
==================================================

Como todas las recomendaciones fueron ejecutadas en Wio:

1. localizar todos los datos reales ya conservados;
2. registrar todos los fills completos;
3. reparar aceptaciones huérfanas;
4. solicitar únicamente los datos reales que falten;
5. no usar las estimaciones para completar huecos;
6. no duplicar movimientos;
7. reconciliar ventas, compras y comisiones como un lote.

Entrega adicional obligatoria:

OWNER_CONFIRMED_ALL_RECOMMENDATIONS_EXECUTED_IN_WIO = TRUE

RECOMMENDATIONS_PENDING_EXECUTION_IN_WIO = 0

RECOMMENDATIONS_PENDING_REGISTRATION_IN_MIZAN =
<número>

EXECUTION_DATA_COMPLETE =
<número>

EXECUTION_DATA_REQUIRED =
<número>

CASH_OVERSPEND_BLOCKS_ON_POST_TRADE_REGISTRATION =
0