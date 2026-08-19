MIZAN · HOTFIX URGENTE
REGISTRO DE EJECUCIONES WIO EN LIBRO MAYOR
CASH_OVERSPEND_BLOCKED · ACEPTACIONES HUÉRFANAS · RECONCILIACIÓN

PRIORIDAD

CRÍTICA.

Omar ya ejecutó en Wio las recomendaciones de la revisión actual.

El objetivo no es ejecutar nuevas operaciones.

El objetivo es:

1. registrar correctamente las ejecuciones reales ya realizadas;
2. evitar cualquier duplicación;
3. corregir el flujo para que una operación ejecutada en el broker no sea
   bloqueada como si todavía fuera una propuesta;
4. asegurar que “Aceptada / Registrada en el Book” solo aparezca después de
   crear correctamente el movimiento del Libro Mayor;
5. reconciliar ventas, compras, comisiones, holdings y cash.

Aparcar completamente Gate 2F.

No modificar código ni schema de La Lente salvo que exista una dependencia
accidental que deba aislarse.

==================================================
0. ACLARACIÓN DEL CASO PANW
==================================================

Recomendación:

ticker =
PANW

acción =
INCORPORAR

cantidad recomendada =
0,342213

precio EOD de referencia =
323,92 USD

importe estimado =
110,85 USD

Ejecución real introducida:

cantidad real =
0,33422822

precio real =
331,51 USD

comisión =
0,52 USD

Cálculo real:

gross_amount =
0,33422822 × 331,51
= 110,7999972122 USD

cash_debit =
gross_amount + commission
= 111,3199972122 USD

cash_debit redondeado =
111,32 USD

Por tanto:

RECOMMENDED_ESTIMATED_AMOUNT =
110,85 USD

ACTUAL_EXECUTED_GROSS_AMOUNT =
110,80 USD

ACTUAL_EXECUTED_TOTAL_CASH =
111,32 USD

El cálculo de 111,32 USD es correcto.

El error consiste en:

- bloquear el registro de un fill que ya ocurrió en Wio;
- informar available cash = 0,00 sin haber reconciliado las ventas ejecutadas;
- o marcar aceptaciones como registradas sin crear el movimiento contable.

No sustituir 111,32 por 110,85.

El Libro debe registrar la ejecución real, no la estimación original.

==================================================
1. RESTRICCIONES
==================================================

No:

- reenviar órdenes a Wio;
- ejecutar operaciones;
- crear operaciones duplicadas;
- modificar cantidades o precios reales introducidos;
- inventar cash;
- inventar aportaciones;
- eliminar movimientos existentes;
- resetear decisiones;
- perder datos introducidos en formularios;
- modificar el Track Papel;
- activar hard cap;
- continuar Gate 2F;
- hacer push;
- crear tag.

Solo están autorizadas escrituras destinadas a:

- registrar fills reales ya ejecutados y confirmados por Omar;
- reparar aceptaciones huérfanas;
- corregir holdings, cash y Libro Mayor como consecuencia directa de esos
  fills;
- conservar auditoría y trazabilidad.

Antes de cualquier reparación productiva:

- backup SQLite;
- SHA-256;
- integrity_check;
- foreign_key_check;
- baseline de Book, movimientos, decisiones, holdings, cash y NAV.

==================================================
2. AUDITAR EL ESTADO ACTUAL ANTES DE TOCAR NADA
==================================================

Identificar la revisión concreta que generó las recomendaciones mostradas.

No asumir únicamente que son 25 filas.

Localizar:

- review_run_id;
- recommendation_id;
- ticker;
- recommendation type;
- recommended quantity;
- reference price;
- estimated amount;
- decision status;
- accepted_at;
- execution form values;
- movement_id;
- Book entry;
- holding mutation;
- cash mutation;
- error;
- idempotency key.

Incluir PANW si pertenece a la misma revisión o a una revisión relacionada que
fue ejecutada por Omar.

Clasificar cada recomendación:

A. PENDING_NOT_RECORDED

- no aceptada;
- sin movimiento.

B. ACCEPTED_AND_RECORDED

- aceptada;
- movimiento existente;
- Book actualizado;
- cash actualizado;
- holding actualizado.

C. ACCEPTED_ORPHANED

- aparece aceptada o “Registrada en el Book”;
- no existe movimiento correspondiente.

D. MOVEMENT_EXISTS_DECISION_NOT_ACCEPTED

- movimiento existente;
- decisión no refleja la aceptación.

E. PARTIAL_WRITE

- existe solo parte de:
  - movimiento;
  - holding;
  - cash;
  - decisión;
  - audit link.

F. DUPLICATE_RISK

- más de un movimiento candidato para la misma ejecución.

Entregar preview completo antes de reparar.

Declarar:

RECOMMENDATIONS_IN_REVIEW = <número>
ACCEPTED_AND_RECORDED = <número>
ACCEPTED_ORPHANED = <número>
MOVEMENT_WITHOUT_ACCEPTANCE = <número>
PARTIAL_WRITES = <número>
DUPLICATE_RISKS = <número>

No reparar automáticamente un caso ambiguo o duplicado.

==================================================
3. IDENTIFICAR EL DEFECTO EXACTO
==================================================

Trazar el flujo actual:

Aceptar recomendación
→ validar ejecución
→ validar cash
→ crear movimiento
→ actualizar Book
→ actualizar holding
→ actualizar cash
→ marcar decisión aceptada
→ mostrar “Registrada en el Book”

Determinar:

- dónde se lanza CASH_OVERSPEND_BLOCKED;
- qué fuente utiliza para available cash;
- si lee cash real, cash pendiente o cero por fallback;
- si incluye ventas aceptadas en la misma revisión;
- si utiliza el importe recomendado en lugar del ejecutado;
- si marca la decisión antes de crear el movimiento;
- si existe más de una conexión/transacción SQLite;
- si un fallo posterior deja la decisión aceptada;
- si la UI muestra éxito basándose solo en decision.status;
- si la comisión se incluye correctamente;
- si las ventas aportan proceeds netos de comisión;
- si los movimientos están vinculados por recommendation_id.

Entregar:

ROOT_CAUSE_CASH_ZERO =
<causa exacta>

ROOT_CAUSE_LEDGER_NOT_WRITTEN =
<causa exacta>

ROOT_CAUSE_FALSE_BOOK_SUCCESS =
<causa exacta>

No aplicar un parche hasta demostrar las tres causas.

==================================================
4. DIFERENCIAR PROPUESTA Y EJECUCIÓN REAL
==================================================

Crear o consolidar dos modos explícitos.

A. PRE_TRADE_PROPOSAL

La operación todavía no ocurrió.

En este modo:

- el sistema puede comprobar capacidad;
- puede bloquear por falta de cash;
- no crea movimiento;
- no modifica Book;
- no modifica holding;
- no modifica cash.

B. BROKER_EXECUTED_FILL

La operación ya fue ejecutada en Wio.

En este modo:

- la cantidad real es la del fill;
- el precio real es el del fill;
- la comisión real es la introducida;
- el movimiento debe registrarse;
- la recomendación sirve como lineage, no como límite del importe;
- la diferencia frente a la estimación debe mostrarse, no bloquearse.

Para BROKER_EXECUTED_FILL:

CASH_OVERSPEND_BLOCKED no puede impedir la contabilización del fill real.

Si el cash interno no reconcilia, el resultado debe ser:

EXECUTION_RECORDED_RECONCILIATION_REQUIRED

No:

CASH_OVERSPEND_BLOCKED

No desactivar el control para operaciones futuras.

==================================================
5. FUENTE DE VERDAD DE LA EJECUCIÓN
==================================================

Para un fill real de compra:

executed_gross_amount =
executed_quantity × executed_price

executed_cash_effect =
-(executed_gross_amount + commission)

Para un fill real de venta:

executed_gross_amount =
executed_quantity × executed_price

executed_cash_effect =
executed_gross_amount - commission

El importe recomendado:

recommended_quantity × reference_price

es únicamente:

ESTIMATED_RECOMMENDATION_AMOUNT

No debe:

- sustituir el importe real;
- limitar el fill real;
- provocar rechazo porque el precio cambió;
- cambiar la cantidad ejecutada;
- cambiar la comisión.

Persistir separadamente:

- recommended quantity;
- reference price;
- estimated recommendation amount;
- executed quantity;
- executed price;
- executed gross amount;
- commission;
- executed cash effect;
- slippage amount;
- slippage percentage;
- execution date;
- source = WIO_MANUAL_CONFIRMATION.

==================================================
6. ENTRADA EDITABLE DEL IMPORTE
==================================================

Omar solicita poder introducir el importe real.

Evitar cuatro campos contradictorios simultáneamente.

Implementar modos de entrada claros.

A. MODO CANTIDAD Y PRECIO

Campos autoritativos:

- executed_quantity;
- executed_price;
- commission.

Derivar:

- gross amount;
- total cash debit/proceeds.

B. MODO IMPORTE TOTAL

Campos autoritativos:

- total cash amount;
- executed_price;
- commission.

Para compra:

derived_quantity =
(total_cash_amount - commission) / executed_price

Para venta, si total cash amount significa proceeds netos:

derived_quantity =
(total_cash_amount + commission) / executed_price

La UI debe indicar inequívocamente si el importe introducido es:

- bruto sin comisión;
- total debitado;
- proceeds netos.

Preferencia:

IMPORTE TOTAL DEBITADO/ABONADO, COMISIÓN INCLUIDA.

No permitir confirmar si los campos autoritativos y derivados no reconcilian
dentro de la tolerancia monetaria.

Mostrar siempre antes de confirmar:

- importe estimado de recomendación;
- importe bruto ejecutado;
- comisión;
- débito/abono total real;
- diferencia frente a estimación.

==================================================
7. PROCESAMIENTO DE VENTAS Y COMPRAS DE LA REVISIÓN
==================================================

Omar indica que ejecutó todas las recomendaciones en Wio.

Para reconciliar la revisión:

1. identificar todas las ventas/reducciones/eliminaciones ejecutadas;
2. identificar todas las compras/aumentos/incorporaciones ejecutadas;
3. obtener de los datos conservados:
   - cantidad real;
   - precio real;
   - comisión;
   - fecha;
4. registrar cronológicamente cuando existan timestamps fiables;
5. si solo existe fecha y no hora:
   - no inventar horas;
   - agrupar mediante execution_batch_id;
   - calcular el efecto neto del batch;
   - mostrar la limitación temporal.

No modificar el orden económico real para simular cash disponible.

Puede calcularse adicionalmente:

same_day_sell_proceeds

same_day_buy_debits

net_batch_cash_effect

Pero no fabricar que una venta ocurrió antes que una compra si no existe
evidencia.

El modelo debe poder registrar el batch real aunque exista un déficit
transitorio de ordenación interna, marcándolo para conciliación.

==================================================
8. CASH Y OPERACIÓN YA EJECUTADA
==================================================

Para operaciones futuras no ejecutadas:

- cash insuficiente puede bloquear.

Para una operación ya ejecutada en el broker:

- debe registrarse el fill;
- debe actualizarse el Libro;
- debe actualizarse el holding;
- debe aplicarse el efecto de cash;
- debe marcarse una diferencia si el modelo interno no reconcilia.

No inventar una aportación para evitar el déficit.

No modificar el importe del fill.

Si después de registrar todo el batch:

book_cash != broker_cash conocido

marcar:

CASH_RECONCILIATION_REQUIRED

y entregar:

- opening book cash;
- total sell proceeds netos;
- total buy debits;
- commissions;
- closing calculated cash;
- broker cash, si se proporcionó;
- difference.

Si no se conoce el broker cash:

- registrar el fill;
- calcular closing book cash;
- marcar BROKER_CASH_NOT_PROVIDED;
- no declarar reconciliación completa.

==================================================
9. ATOMICIDAD DEL REGISTRO
==================================================

Para cada fill real, una única transacción SQLite debe:

1. validar recommendation;
2. validar idempotencia;
3. insertar movimiento en Libro Mayor;
4. aplicar efecto de cash;
5. actualizar o crear holding;
6. vincular movement_id a recommendation/decision;
7. persistir execution details;
8. marcar decisión aceptada;
9. registrar audit event;
10. COMMIT.

La decisión solo puede pasar a:

ACCEPTED

después de crear correctamente el movimiento y actualizar el estado económico.

Si falla cualquier paso:

- ROLLBACK;
- decisión permanece pendiente;
- no mostrar “Registrada en el Book”;
- conservar los datos del formulario;
- mostrar error real.

Debe cumplirse:

DECISION_ACCEPTED
⇔
LEDGER_MOVEMENT_EXISTS_AND_LINKED

No permitir aceptaciones huérfanas nuevas.

==================================================
10. ESTADO DE ÉXITO EN LA UI
==================================================

Mostrar:

“Aceptada · Registrada en el Libro Mayor”

únicamente cuando:

- movement_id existe;
- movement está committed;
- execution está vinculada;
- holding está actualizado;
- cash effect está registrado.

Si el fill fue registrado pero existe diferencia de cash:

“Aceptada · Registrada · Reconciliación de cash pendiente”

No mostrar:

“Registrada en el Book”

si solo se actualizó la decisión.

La UI debe mostrar el movement_id o una referencia auditable.

==================================================
11. REPARACIÓN DE ACEPTACIONES HUÉRFANAS
==================================================

Crear una herramienta one-shot idempotente:

repairAcceptedRecommendationsWithoutMovement

Debe:

1. localizar ACCEPTED_ORPHANED;
2. recuperar los valores reales conservados;
3. mostrar preview;
4. calcular importes;
5. buscar movimientos equivalentes existentes;
6. impedir duplicados;
7. crear el movimiento faltante;
8. actualizar holding y cash;
9. enlazar movement_id;
10. registrar repair audit event.

No reparar si faltan:

- cantidad real;
- precio real;
- fecha;
- acción;
- ticker.

En esos casos:

MANUAL_EXECUTION_DATA_REQUIRED

No utilizar los valores recomendados como si fueran la ejecución real.

==================================================
12. RECUPERACIÓN DE DATOS DEL FORMULARIO
==================================================

El error indica:

“datos conservados, puedes reintentar”.

Verificar dónde se conservaron:

- base de datos;
- session storage;
- local storage;
- estado frontend;
- audit event;
- request payload log.

Antes de reiniciar servicio o modificar UI:

- exportar los datos conservados;
- asociarlos a recommendation_id;
- calcular hash;
- evitar que se pierdan.

Entregar:

SAVED_EXECUTION_FORMS_FOUND = <número>
SAVED_EXECUTION_FORMS_COMPLETE = <número>
SAVED_EXECUTION_FORMS_INCOMPLETE = <número>

==================================================
13. RECUPERACIÓN DEL BATCH EJECUTADO EN WIO
==================================================

Crear preview de reparación para la revisión completa.

Por ticker:

- recommendation_id;
- action;
- recommended quantity;
- reference price;
- estimated amount;
- actual quantity;
- actual price;
- commission;
- execution date;
- actual gross;
- actual cash effect;
- current decision status;
- existing movement;
- repair action;
- duplicate risk;
- reconciliation status.

No ejecutar reparación sin entregar primero el preview.

La reparación debe ser idempotente.

Segunda ejecución:

- ALREADY_REPAIRED;
- cero movimientos nuevos;
- cero cambios económicos.

==================================================
14. PANW
==================================================

Para PANW, conservar exactamente:

executed_quantity =
0,33422822

executed_price =
331,51

commission =
0,52

execution_date =
2026-07-24

executed_gross =
110,7999972122

executed_total_cash_debit =
111,3199972122

No cambiarlo a:

quantity =
0,342213

ni:

amount =
110,85

salvo que Omar indique expresamente que los datos reales introducidos eran
incorrectos.

Si PANW todavía no fue aceptada por el bloqueo:

- registrar como fill real pendiente de reparación;
- no reenviar a Wio;
- no crear una segunda operación.

==================================================
15. PRUEBAS
==================================================

Crear:

verify-real-execution-amount-calculation.mjs
verify-real-execution-vs-recommendation.mjs
verify-real-execution-ledger-atomicity.mjs
verify-real-execution-cash-posting.mjs
verify-real-execution-sell-proceeds.mjs
verify-real-execution-commission.mjs
verify-real-execution-idempotency.mjs
verify-real-execution-orphan-repair.mjs
verify-real-execution-batch-reconciliation.mjs
verify-real-execution-ui-status.mjs

Casos obligatorios:

A. PANW calcula 111,32.
B. Diferencia frente a 110,85 no bloquea fill.
C. Pre-trade sin cash sí puede bloquear.
D. Fill ejecutado se registra con warning de reconciliación.
E. Compra reduce cash por gross + commission.
F. Venta aumenta cash por gross - commission.
G. Decisión no se acepta si falla movimiento.
H. Movimiento no se duplica por retry.
I. Aceptación huérfana se repara una vez.
J. Mismo recommendation/fill no crea dos movimientos.
K. Ventas y compras de batch reconcilian.
L. UI no muestra éxito falso.
M. Valores del formulario se conservan tras error.
N. Holdings coinciden con cantidades ejecutadas.
O. Comisiones coinciden con ejecuciones.
P. Book y Libro Mayor quedan enlazados.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- financial core;
- Book;
- movimientos;
- cash;
- NAV;
- recommendation acceptance;
- repository integrity.

==================================================
16. PRODUCCIÓN
==================================================

Antes:

- detener escrituras;
- backup;
- SHA-256;
- integrity_check;
- foreign_key_check;
- baseline;
- exportar datos conservados;
- preview completo;
- tests verdes.

Aplicar primero el hotfix de atomicidad.

Después:

- reparar aceptaciones huérfanas;
- registrar fills pendientes confirmados por Omar;
- reconciliar el batch;
- reiniciar una sola instancia;
- health;
- env-info;
- comprobar Libro Mayor;
- comprobar Book;
- comprobar holdings;
- comprobar cash;
- comprobar NAV.

No ejecutar ninguna orden en Wio.

==================================================
17. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = <hash actual>
CANONICAL_HEAD_AFTER = <hash>

ROOT_CAUSE_CASH_ZERO = <causa>
ROOT_CAUSE_LEDGER_NOT_WRITTEN = <causa>
ROOT_CAUSE_FALSE_BOOK_SUCCESS = <causa>

RECOMMENDATIONS_IN_REVIEW = <número>
SAVED_EXECUTION_FORMS_FOUND = <número>

ACCEPTED_AND_RECORDED_BEFORE = <número>
ACCEPTED_ORPHANED_BEFORE = <número>
PARTIAL_WRITES_BEFORE = <número>
DUPLICATE_RISKS = <número>

BROKER_EXECUTED_FILL_MODE = PASS/FAIL
PRE_TRADE_CASH_GUARD_PRESERVED = PASS/FAIL

PANW_RECOMMENDED_ESTIMATE = 110,85
PANW_EXECUTED_GROSS = 110,80
PANW_COMMISSION = 0,52
PANW_EXECUTED_TOTAL_CASH = 111,32
PANW_REGISTERED = PASS/FAIL
PANW_MOVEMENT_ID = <id>

ORPHANED_ACCEPTANCES_REPAIRED = <número>
PENDING_FILLS_RECORDED = <número>
MOVEMENTS_CREATED = <número>
DUPLICATES_CREATED = 0

SELL_PROCEEDS_NET = <importe>
BUY_DEBITS_TOTAL = <importe>
COMMISSIONS_TOTAL = <importe>
NET_BATCH_CASH_EFFECT = <importe>

BOOK_CASH_BEFORE = <importe>
BOOK_CASH_AFTER = <importe>
BROKER_CASH = <importe/NOT_PROVIDED>
CASH_RECONCILIATION_DIFFERENCE = <importe/UNKNOWN>
CASH_RECONCILIATION_STATUS =
PASS /
REQUIRED /
BROKER_CASH_NOT_PROVIDED

DECISIONS_WITHOUT_MOVEMENT_AFTER = 0
MOVEMENTS_WITHOUT_DECISION_AFTER = 0
PARTIAL_WRITES_AFTER = 0

HOLDINGS_RECONCILED = PASS/FAIL
BOOK_RECONCILED = PASS/FAIL
LEDGER_RECONCILED = PASS/FAIL
CASH_RECONCILED = PASS/FAIL/NOT_VERIFIABLE
NAV_RECALCULATED = PASS/FAIL

NO_WIO_ORDERS_SENT = PASS/FAIL
NO_DUPLICATE_EXECUTIONS = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

LOCAL_COMMIT = <hash>

Detenerse después del hotfix y la reconciliación.