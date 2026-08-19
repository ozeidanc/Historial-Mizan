MIZAN · HOTFIX CRÍTICO
REGISTRO POST-TRADE DE EJECUCIONES WIO
REPARACIÓN GLOBAL DEL LOTE · LIBRO MAYOR · HOLDINGS · CASH

PRIORIDAD

CRÍTICA.

Aparcar completamente Gate 2F hasta cerrar este hotfix.

==================================================
0. CONTRATO FUNCIONAL CONFIRMADO POR OMAR
==================================================

Omar confirma que el funcionamiento real es siempre:

1. Mizan formula una recomendación;
2. Omar ejecuta manualmente la operación en Wio;
3. después abre la recomendación en Mizan;
4. introduce cantidad, precio, comisión y fecha reales;
5. pulsa Aceptar;
6. Mizan registra la ejecución ya realizada en el Libro Mayor.

Por tanto:

OWNER_CONFIRMED_ALL_RECOMMENDATIONS_EXECUTED_IN_WIO =
TRUE

El formulario de aceptación es exclusivamente POST-TRADE.

Aceptar significa:

REGISTRAR EJECUCIÓN REAL YA REALIZADA EN WIO

No significa:

- autorizar una operación futura;
- comprobar si Mizan habría permitido comprar;
- enviar una orden a Wio;
- reservar cash;
- simular una operación;
- aceptar los valores estimados de la recomendación.

Todas las recomendaciones del lote fueron ejecutadas en Wio.

Una recomendación todavía pendiente en Mizan significa:

EXECUTED_IN_WIO_PENDING_REGISTRATION_IN_MIZAN

No significa:

NOT_EXECUTED_IN_WIO

==================================================
1. INCIDENTE
==================================================

Al introducir fills reales, Mizan bloquea múltiples recomendaciones con:

CASH_OVERSPEND_BLOCKED
requiere <gross + commission>
disponible 0,00

No es un error específico de PANW.

Es un defecto sistémico del flujo de aceptación de recomendaciones.

Mizan está aplicando una validación pre-trade de cash a operaciones que ya
fueron ejecutadas en Wio y que ahora deben registrarse contablemente.

También existen indicios de que algunas recomendaciones pueden mostrar:

Aceptada
Registrada en el Book

sin que se haya creado correctamente el movimiento correspondiente en el
Libro Mayor.

==================================================
2. OBJETIVOS
==================================================

Corregir el flujo global para que todas las ejecuciones reales de Wio:

- se registren con la cantidad real;
- se registren con el precio real;
- conserven el importe bruto real;
- registren la comisión separadamente;
- apliquen el efecto real sobre cash;
- creen un movimiento en el Libro Mayor;
- actualicen el holding;
- queden vinculadas a la recomendación;
- actualicen la decisión solo después del COMMIT;
- no se dupliquen;
- no sean bloqueadas por CASH_OVERSPEND_BLOCKED;
- puedan reconciliarse como un lote;
- conserven trazabilidad completa.

La corrección debe aplicar a:

- INCORPORAR;
- AUMENTAR;
- REDUCIR;
- ELIMINAR;
- cualquier otra recomendación que represente un fill ejecutado en Wio.

No crear excepciones por ticker.

No hardcodear PANW.

==================================================
3. RESTRICCIONES OPERATIVAS
==================================================

No:

- enviar órdenes a Wio;
- volver a ejecutar operaciones;
- asumir que una recomendación pendiente no fue ejecutada;
- borrar decisiones;
- borrar movimientos;
- borrar formularios conservados;
- modificar cantidades reales;
- modificar precios reales;
- modificar comisiones reales;
- sustituir datos reales por estimaciones;
- crear movimientos duplicados;
- inventar aportaciones;
- inventar cash;
- inventar horas de ejecución;
- continuar Gate 2F;
- activar MANUAL_NOTIONAL_HARD_CAP;
- hacer push;
- crear tag.

Antes de cualquier escritura productiva:

- detener escrituras;
- confirmar una sola instancia;
- crear backup SQLite;
- calcular SHA-256;
- abrir y validar el backup;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- capturar baseline de Book;
- capturar baseline de movimientos;
- capturar baseline de holdings;
- capturar baseline de cash;
- capturar baseline de NAV;
- capturar baseline de decisiones;
- exportar todos los datos de formularios conservados;
- generar preview completo;
- detectar duplicados.

==================================================
4. BASELINE DE CÓDIGO
==================================================

Determinar el HEAD actual real antes de modificar.

El último HEAD conocido antes de aparcar Gate 2F es:

88892cd

Pero no asumirlo si existe trabajo posterior.

Entregar:

CANONICAL_HEAD_FOUND =
<hash>

SCHEMA_FOUND =
<versión>

GIT_STATUS_BEFORE =
<estado>

UNTRACKED_FILES =
<lista>

No descartar ni mezclar cambios incompletos de otro gate.

El hotfix debe quedar aislado del trabajo de Gate 2F.

==================================================
5. IDENTIFICAR EL LOTE COMPLETO
==================================================

Localizar la revisión o revisiones que generaron todas las recomendaciones
ejecutadas por Omar.

La tabla mostrada contiene al menos 25 recomendaciones sobre posiciones
existentes.

PANW aparece en un formulario de INCORPORAR y puede pertenecer:

- al mismo review run;
- a otra sección del mismo run;
- a otra recomendación relacionada.

Resolver mediante IDs y lineage, no mediante ticker solamente.

Inventariar:

- review_run_id;
- recommendation_id;
- ticker;
- action;
- recommended_quantity;
- reference_price;
- estimated_amount;
- decision_status;
- accepted_at;
- saved execution fields;
- existing execution detail;
- existing movement_id;
- holding mutation;
- cash mutation;
- audit event;
- error;
- idempotency key.

No asumir un número fijo de recomendaciones.

Entregar:

REVIEW_RUNS_FOUND =
<lista>

RECOMMENDATIONS_IN_SCOPE =
<número>

==================================================
6. TODAS ESTÁN EJECUTADAS EN WIO
==================================================

La clasificación debe partir de esta confirmación:

RECOMMENDATIONS_PENDING_EXECUTION_IN_WIO =
0

Clasificar cada recomendación en:

A. EXECUTED_PENDING_REGISTRATION_DATA_COMPLETE

- fue ejecutada en Wio;
- existen cantidad real;
- precio real;
- comisión;
- fecha;
- no existe movimiento completo en Mizan.

B. EXECUTED_PENDING_REGISTRATION_DATA_REQUIRED

- fue ejecutada en Wio;
- faltan uno o más valores reales;
- no puede registrarse todavía;
- debe solicitarse el dato exacto a Omar.

C. ACCEPTED_AND_RECORDED

- movimiento existente;
- execution detail existente;
- recomendación vinculada;
- holding actualizado;
- efecto de cash contabilizado;
- decisión aceptada.

D. ACCEPTED_ORPHANED

- figura aceptada;
- no existe movimiento vinculado.

E. MOVEMENT_WITHOUT_ACCEPTANCE

- existe movimiento;
- la decisión no refleja el registro.

F. PARTIAL_WRITE

Existe solo parte de:

- execution detail;
- movimiento;
- holding;
- cash;
- decisión;
- lineage.

G. DUPLICATE_RISK

- existen dos o más movimientos candidatos;
- o el fill puede haber sido registrado mediante otra ruta.

No clasificar ninguna recomendación del lote como:

NOT_EXECUTED_IN_WIO

==================================================
7. RECUPERAR LOS DATOS INTRODUCIDOS
==================================================

El sistema mostró:

“datos conservados, puedes reintentar”.

Localizar esos datos en:

- base de datos;
- drafts;
- audit events;
- session storage;
- local storage;
- estado persistido del frontend;
- request logs;
- payloads de error;
- registros temporales;
- cualquier mecanismo real utilizado por la aplicación.

Antes de reiniciar o desplegar:

- exportar todos los datos encontrados;
- asociarlos con recommendation_id;
- asociarlos con ticker;
- calcular un hash por payload;
- conservar una copia de recuperación;
- no alterar sus valores.

Por formulario recuperar:

- recommendation_id;
- ticker;
- action;
- executed_quantity;
- executed_price;
- commission;
- execution_date;
- execution_time nullable;
- broker_reference nullable;
- saved_at;
- error;
- payload_hash.

Entregar:

SAVED_EXECUTION_FORMS_FOUND =
<número>

SAVED_EXECUTION_FORMS_COMPLETE =
<número>

SAVED_EXECUTION_FORMS_INCOMPLETE =
<número>

SAVED_EXECUTION_FORMS_UNLINKED =
<número>

No utilizar los valores recomendados para rellenar datos reales ausentes.

==================================================
8. SEMÁNTICA CONTABLE OBLIGATORIA
==================================================

Para una COMPRA:

executed_gross_amount =
executed_quantity × executed_price

commission_amount =
commission

executed_cash_effect =
-(executed_gross_amount + commission_amount)

Para una VENTA:

executed_gross_amount =
executed_quantity × executed_price

commission_amount =
commission

executed_cash_effect =
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
- broker_reference nullable;
- execution_source = WIO_MANUAL_CONFIRMATION;
- slippage_amount;
- slippage_percentage.

No guardar:

gross + commission

como importe comprado.

No guardar:

gross - commission

como importe vendido.

El principal del movimiento es:

executed_gross_amount

La comisión es:

commission_amount

El efecto total sobre cash es:

executed_cash_effect

==================================================
9. CASO PANW COMO PRUEBA, NO COMO EXCEPCIÓN
==================================================

PANW fue ejecutada con:

executed_quantity =
0.33422822

executed_price =
331.51

commission =
0.52

execution_date =
2026-07-24

Cálculo:

executed_gross_amount =
0.33422822 × 331.51
=
110.7999972122

Visualización:

executed_gross_amount_display =
110.80

Efecto sobre cash:

executed_cash_effect =
-(110.7999972122 + 0.52)
=
-111.3199972122

Visualización:

total_cash_debit_display =
111.32

La recomendación original:

recommended_quantity =
0.342213

reference_price =
323.92

estimated_recommendation_amount =
110.85 aproximadamente

es solo una estimación.

Registro correcto:

PANW_LEDGER_PRINCIPAL =
110.80

PANW_LEDGER_COMMISSION =
0.52

PANW_CASH_DEBIT =
111.32

PANW_HOLDING_QUANTITY_ADDED =
0.33422822

No sustituir la ejecución real por:

- cantidad 0.342213;
- importe 110.85;
- precio 323.92.

La misma regla debe aplicarse a todas las operaciones.

==================================================
10. CONTRATO DEL FORMULARIO
==================================================

El formulario de aceptación de recomendaciones es siempre post-trade.

No implementar selector:

- Todavía no ejecutada;
- Ya ejecutada en Wio.

No existe modo PRE_TRADE dentro de esta pantalla.

Al abrir el formulario debe mostrarse:

EJECUCIÓN REAL EN WIO

Campos autoritativos:

- cantidad realmente ejecutada;
- precio real de ejecución;
- comisión;
- fecha de ejecución;
- hora opcional;
- referencia Wio opcional.

Valores derivados:

- importe bruto;
- débito total para compra;
- abono neto para venta;
- diferencia frente a recomendación.

Mostrar separadamente:

RECOMENDACIÓN ORIGINAL

- cantidad recomendada;
- precio EOD de referencia;
- importe estimado.

Botón recomendado:

REGISTRAR EJECUCIÓN WIO

o:

ACEPTAR Y REGISTRAR EJECUCIÓN REAL

==================================================
11. CASH_OVERSPEND_BLOCKED
==================================================

Eliminar:

CASH_OVERSPEND_BLOCKED

de la ruta de aceptación y registro post-trade.

La operación ya ocurrió.

Mizan no puede rechazar registrar la realidad porque su cash interno no esté
reconciliado.

Si:

required_cash > internal_available_cash

el flujo debe:

1. validar el fill;
2. registrar execution detail;
3. registrar movimiento;
4. actualizar holding;
5. aplicar efecto de cash;
6. vincular recommendation;
7. aceptar decisión;
8. registrar audit event;
9. marcar reconciliación pendiente.

Resultado:

EXECUTION_RECORDED_RECONCILIATION_REQUIRED

No:

CASH_OVERSPEND_BLOCKED

Si existe otro flujo realmente pre-trade fuera de esta pantalla:

- conservar allí su control de cash;
- no desactivarlo globalmente;
- aislar las dos rutas.

Pero el formulario actual de aceptación es siempre post-trade.

Resultado obligatorio:

CASH_OVERSPEND_BLOCKS_ON_POST_TRADE_REGISTRATION =
0

==================================================
12. EL ORDEN DE REGISTRO NO ES EL ORDEN DE WIO
==================================================

Omar puede introducir las recomendaciones en Mizan en un orden diferente del
orden en que Wio ejecutó las operaciones.

Por tanto, Mizan no puede:

- bloquear una compra porque todavía no se registró una venta;
- asumir que el orden de clics es el orden del broker;
- exigir que el cash interno cuadre fill por fill antes de registrar;
- inventar el orden dentro del día.

Cada fill real debe registrarse atómicamente.

Después debe reconciliarse el lote completo.

Si existe execution_time real:

- utilizarlo.

Si solo existe execution_date:

- agrupar por execution_batch_id;
- no inventar una hora;
- registrar ORDER_WITHIN_DAY_UNKNOWN.

==================================================
13. ROOT CAUSE OBLIGATORIO
==================================================

Antes del parche definitivo, identificar:

ROOT_CAUSE_CASH_ZERO

Investigar si:

- las ventas ejecutadas no se registraron;
- sus proceeds no aumentaron cash;
- la fuente de cash es incorrecta;
- existe fallback a cero;
- se usa cash disponible previo a registrar el lote;
- la lectura ocurre antes del COMMIT;
- las comisiones causan una diferencia;
- el cash inicial estaba desactualizado;
- Book y aceptación usan fuentes distintas.

ROOT_CAUSE_LEDGER_NOT_WRITTEN

Investigar si:

- CASH_OVERSPEND_BLOCKED ocurre antes del INSERT;
- se marca decisión en una transacción diferente;
- el movimiento falla después de aceptar;
- existe rollback parcial;
- se escribe en una DB distinta;
- falta recommendation_id en el movimiento;
- Book lee otra tabla;
- el endpoint responde éxito antes del COMMIT.

ROOT_CAUSE_FALSE_BOOK_SUCCESS

Investigar si la UI muestra:

Aceptada
Registrada en el Book

basándose solo en:

decision.status = ACCEPTED

en vez de comprobar:

movement_id exists
AND movement committed
AND holding updated
AND cash effect posted

Entregar las tres causas concretas.

No limitarse a cambiar el mensaje de error.

==================================================
14. ATOMICIDAD POR FILL
==================================================

Cada fill real debe registrarse dentro de una única transacción SQLite:

1. cargar recommendation;
2. comprobar que pertenece al lote;
3. comprobar idempotencia;
4. detectar movimientos equivalentes;
5. validar executed_quantity;
6. validar executed_price;
7. validar commission;
8. validar execution_date;
9. calcular gross amount;
10. calcular cash effect;
11. insertar execution detail;
12. insertar movimiento en Libro Mayor;
13. aplicar efecto de cash;
14. actualizar o crear holding;
15. vincular movement_id con recommendation;
16. registrar broker execution reference;
17. marcar decisión ACCEPTED;
18. insertar audit event;
19. COMMIT.

La decisión solo puede pasar a ACCEPTED después de que:

- el movimiento exista;
- el holding esté actualizado;
- el efecto de cash esté registrado;
- el vínculo esté creado.

Invariante:

DECISION_ACCEPTED
SI Y SOLO SI
LEDGER_MOVEMENT_COMMITTED_AND_LINKED

Si falla cualquier paso:

- ROLLBACK completo;
- decisión permanece pendiente de registro;
- cero movimiento parcial;
- cero cambio parcial del holding;
- cero cambio parcial de cash;
- datos del formulario permanecen conservados;
- UI muestra el error real;
- UI no muestra “Registrada”.

==================================================
15. CASH Y CONCILIACIÓN
==================================================

El cash interno no autoriza ni rechaza una ejecución pasada.

Es un resultado contable que debe reconciliarse con Wio.

No inventar una aportación para evitar un saldo negativo o una diferencia.

No modificar el fill.

No modificar su comisión.

No alterar el precio.

Si registrar temporalmente un fill produce un cash interno negativo o
inconsistente:

- conservar el efecto económico real;
- registrar CASH_RECONCILIATION_REQUIRED;
- no ocultar el déficit;
- no redondearlo a cero;
- no afirmar que está reconciliado.

La conciliación final debe realizarse sobre el lote completo.

==================================================
16. RECONCILIACIÓN DEL LOTE
==================================================

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

Si Omar proporciona el cash final de Wio:

cash_reconciliation_difference =
broker_closing_cash - closing_calculated_book_cash

Si no proporciona el cash final:

CASH_RECONCILIATION_STATUS =
BROKER_CASH_NOT_PROVIDED

No declarar:

CASH_RECONCILED

sin evidencia del saldo del broker o una fuente equivalente.

Registrar de forma durable:

- execution_batch_id;
- operaciones incluidas;
- opening cash;
- ventas brutas;
- comisiones de venta;
- proceeds netos;
- compras brutas;
- comisiones de compra;
- débitos totales;
- closing calculated cash;
- broker cash nullable;
- diferencia nullable;
- reconciliation status.

==================================================
17. PREVIEW GLOBAL SIN ESCRITURAS
==================================================

Crear:

previewBrokerExecutedFillBatch

Debe mostrar por recomendación:

- review_run_id;
- recommendation_id;
- ticker;
- action;
- recommended_quantity;
- reference_price;
- estimated_amount;
- executed_quantity;
- executed_price;
- gross_amount;
- commission;
- cash_effect;
- execution_date;
- execution_time nullable;
- current decision status;
- existing movement;
- movement_id nullable;
- duplicate risk;
- missing data;
- proposed action;
- reconciliation status.

No escribir.

El preview debe clasificar:

- READY_TO_RECORD;
- ALREADY_RECORDED;
- ACCEPTED_ORPHANED;
- PARTIAL_WRITE;
- EXECUTION_DATA_REQUIRED;
- DUPLICATE_REVIEW_REQUIRED.

Entregar el preview completo antes de reparar Producción.

No reparar casos ambiguos automáticamente.

==================================================
18. REGISTRO CONTROLADO DEL LOTE
==================================================

Crear:

recordBrokerExecutedFillBatch

Solo debe procesar fills:

- confirmados por Omar;
- con datos completos;
- sin ambigüedad;
- sin movimiento equivalente;
- con recommendation lineage resuelto.

Preferencia:

- transacción atómica por fill;
- batch reconciliation durable;
- continuar con otros fills si uno requiere datos manuales;
- no dejar escrituras parciales dentro de un fill.

Para cada fill registrado:

- conservar valores exactos;
- crear movement_id;
- actualizar holding;
- aplicar cash;
- aceptar decisión;
- enlazar recommendation;
- registrar resultado.

Segunda ejecución:

- ALREADY_RECORDED;
- cero movimientos nuevos;
- cero cambios económicos duplicados.

==================================================
19. REPARACIÓN DE ACEPTACIONES HUÉRFANAS
==================================================

Crear:

repairAcceptedRecommendationsWithoutMovement

Debe:

1. localizar ACCEPTED_ORPHANED;
2. recuperar los valores reales conservados;
3. detectar movimientos equivalentes;
4. verificar duplicados;
5. crear execution detail faltante;
6. crear movimiento faltante;
7. aplicar cash;
8. actualizar holding;
9. vincular movement_id;
10. registrar repair audit event.

No usar datos recomendados como ejecución real.

Si falta cualquier valor real obligatorio:

MANUAL_EXECUTION_DATA_REQUIRED

No desaceptar silenciosamente.

No eliminar historia.

Segunda ejecución:

ALREADY_REPAIRED

==================================================
20. DUPLICADOS E IDEMPOTENCIA
==================================================

Crear una idempotency key durable usando:

- recommendation_id;
- action;
- ticker;
- execution_date;
- execution_time nullable;
- executed_quantity;
- executed_price;
- commission;
- broker_reference nullable.

Antes de insertar buscar coincidencias por:

- recommendation_id;
- ticker;
- side;
- date;
- quantity;
- price;
- gross;
- commission;
- broker reference;
- movement lineage.

Mismo fill:

ALREADY_RECORDED

Misma recomendación con datos distintos:

EXECUTION_DATA_CONFLICT

Varios movimientos candidatos:

DUPLICATE_REVIEW_REQUIRED

No crear duplicados.

==================================================
21. ESTADO DE ÉXITO EN UI
==================================================

Mostrar:

Aceptada · Registrada en el Libro Mayor

únicamente cuando:

- movement_id existe;
- transaction committed;
- execution detail existe;
- holding actualizado;
- cash effect contabilizado;
- recommendation vinculada.

Mostrar también:

movement_id =
<referencia>

Si existe diferencia de cash:

Aceptada · Registrada · Conciliación de cash pendiente

No mostrar éxito solo porque:

decision.status = ACCEPTED

No mostrar:

Registrada en el Book

antes del COMMIT.

==================================================
22. DATOS REALES QUE FALTEN
==================================================

Como todas las operaciones ocurrieron en Wio, una recomendación sin datos
completos debe quedar:

EXECUTION_DATA_REQUIRED

Solicitar únicamente:

- cantidad ejecutada;
- precio ejecutado;
- comisión;
- fecha;
- hora opcional;
- referencia Wio opcional.

No pedir a Omar que vuelva a ejecutar la operación.

No usar:

- cantidad recomendada;
- precio EOD;
- importe estimado;

como sustitutos.

==================================================
23. PRUEBAS
==================================================

Crear:

verify-wio-post-trade-registration.mjs
verify-wio-fill-gross-amount.mjs
verify-wio-fill-commission-separation.mjs
verify-wio-fill-cash-effect.mjs
verify-wio-fill-no-cash-block.mjs
verify-wio-fill-ledger-atomicity.mjs
verify-wio-fill-holding-update.mjs
verify-wio-fill-batch-reconciliation.mjs
verify-wio-fill-orphan-repair.mjs
verify-wio-fill-idempotency.mjs
verify-wio-fill-ui-status.mjs
verify-wio-fill-form-recovery.mjs

Casos obligatorios:

A. PANW gross = 110,80.
B. PANW commission = 0,52.
C. PANW debit = 111,32.
D. PANW quantity added = 0,33422822.
E. Diferencia con estimación 110,85 no bloquea.
F. Otra compra afectada utiliza la misma regla.
G. Venta registra gross y proceeds netos separadamente.
H. Post-trade fill con cash cero se registra.
I. Post-trade fill queda marcado para conciliación.
J. No existe CASH_OVERSPEND_BLOCKED en aceptación post-trade.
K. Venta del lote aumenta cash.
L. Compra del lote reduce cash por gross + comisión.
M. Orden de formularios no bloquea fills.
N. Decisión no puede aceptarse sin movimiento.
O. Fallo produce rollback completo.
P. Aceptación huérfana se repara una vez.
Q. Retry no crea duplicados.
R. Holdings usan cantidades reales.
S. Comisiones quedan separadas.
T. UI muestra movement_id.
U. Datos se conservan tras error.
V. Batch genera conciliación durable.
W. Estimaciones nunca sustituyen fills reales.
X. Todas las recomendaciones se consideran ejecutadas en Wio.
Y. Operaciones sin datos quedan EXECUTION_DATA_REQUIRED.
Z. No se envía ninguna orden a Wio.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- financial core;
- Book;
- Libro Mayor;
- movimientos;
- holdings;
- cash;
- NAV;
- recommendation acceptance;
- repository integrity.

==================================================
24. PREVIEW DE PRODUCCIÓN
==================================================

Antes de desplegar:

- crear backup;
- calcular SHA-256;
- validar backup;
- integrity_check;
- foreign_key_check;
- exportar formularios;
- capturar baseline;
- ejecutar hotfix en copia exacta;
- generar preview del lote;
- revisar duplicados;
- revisar huérfanos;
- revisar partial writes;
- ejecutar pruebas;
- iniciar servidor contra la copia;
- comprobar UI.

No continuar si:

- existen duplicados ambiguos;
- se pierden datos del formulario;
- la decisión puede aceptarse sin movimiento;
- PANW no calcula correctamente;
- algún fill se modifica;
- Book, holding y cash no se actualizan atómicamente.

==================================================
25. DESPLIEGUE Y REPARACIÓN
==================================================

Orden obligatorio:

1. detener escrituras;
2. backup productivo;
3. SHA-256;
4. integrity_check;
5. foreign_key_check;
6. baseline;
7. desplegar corrección de atomicidad;
8. desplegar registro post-trade;
9. desplegar estado UI correcto;
10. reiniciar una sola instancia;
11. health;
12. env-info;
13. generar preview del lote;
14. registrar fills completos;
15. reparar aceptaciones huérfanas;
16. dejar datos incompletos como EXECUTION_DATA_REQUIRED;
17. reconciliar el lote;
18. verificar Libro Mayor;
19. verificar movimientos;
20. verificar holdings;
21. verificar cash;
22. recalcular y verificar NAV;
23. comprobar cero duplicados;
24. comprobar cero órdenes enviadas a Wio.

No enviar operaciones al broker.

==================================================
26. VERIFICACIÓN POSTERIOR
==================================================

Después de reparar, cada recomendación debe tener un estado coherente.

ALREADY_RECORDED:

- movimiento existente;
- holding correcto;
- cash aplicado;
- decisión aceptada.

RECORDED_RECONCILIATION_REQUIRED:

- movimiento existente;
- holding correcto;
- cash aplicado;
- decisión aceptada;
- diferencia de cash visible.

EXECUTION_DATA_REQUIRED:

- confirmada como ejecutada en Wio;
- faltan datos reales;
- no se inventó movimiento.

No deben quedar:

- decisiones aceptadas sin movement_id;
- movimientos sin recommendation lineage;
- cambios de holding sin movimiento;
- cambios de cash sin movimiento;
- fills completos bloqueados;
- duplicados.

==================================================
27. CÓDIGO Y COMMIT
==================================================

Añadir únicamente:

- registro post-trade de fills Wio;
- separación gross/commission/cash effect;
- atomicidad;
- idempotencia;
- recuperación de formularios;
- preview de batch;
- registro de batch;
- reparación de huérfanos;
- reconciliación;
- estado UI correcto;
- pruebas;
- documentación del incidente.

No añadir:

- integración automática con Wio;
- envío de órdenes;
- Gate 2F;
- hard cap;
- cambios del Track Papel;
- datos ficticios;
- scratchpads;
- backups;
- bases;
- secretos.

Ejecutar secret scan.

Crear un único commit local:

fix(book): register executed Wio fills without pre-trade cash blocking

No crear tag.
No hacer push.

==================================================
28. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE =
<hash>

CANONICAL_HEAD_AFTER =
<hash>

SCHEMA_BEFORE =
<versión>

SCHEMA_AFTER =
<versión>

OWNER_CONFIRMED_ALL_RECOMMENDATIONS_EXECUTED_IN_WIO =
TRUE

RECOMMENDATIONS_PENDING_EXECUTION_IN_WIO =
0

ROOT_CAUSE_CASH_ZERO =
<causa exacta>

ROOT_CAUSE_LEDGER_NOT_WRITTEN =
<causa exacta>

ROOT_CAUSE_FALSE_BOOK_SUCCESS =
<causa exacta>

REVIEW_RUNS_FOUND =
<lista>

RECOMMENDATIONS_IN_SCOPE =
<número>

SAVED_EXECUTION_FORMS_FOUND =
<número>

SAVED_EXECUTION_FORMS_COMPLETE =
<número>

SAVED_EXECUTION_FORMS_INCOMPLETE =
<número>

SAVED_EXECUTION_FORMS_UNLINKED =
<número>

EXECUTED_PENDING_REGISTRATION_DATA_COMPLETE_BEFORE =
<número>

EXECUTION_DATA_REQUIRED_BEFORE =
<número>

ACCEPTED_AND_RECORDED_BEFORE =
<número>

ACCEPTED_ORPHANED_BEFORE =
<número>

MOVEMENT_WITHOUT_ACCEPTANCE_BEFORE =
<número>

PARTIAL_WRITES_BEFORE =
<número>

DUPLICATE_RISKS =
<número>

POST_TRADE_REGISTRATION_MODE =
PASS/FAIL

CASH_OVERSPEND_BLOCKS_ON_POST_TRADE_REGISTRATION =
<número>

COMMISSION_SEPARATED_FROM_PRINCIPAL =
PASS/FAIL

DECISION_LEDGER_ATOMICITY =
PASS/FAIL

FILLS_RECORDED =
<número>

ORPHANED_ACCEPTANCES_REPAIRED =
<número>

PARTIAL_WRITES_REPAIRED =
<número>

EXECUTION_DATA_REQUIRED_AFTER =
<número>

DUPLICATES_CREATED =
0

TOTAL_BUY_GROSS =
<importe>

TOTAL_BUY_COMMISSIONS =
<importe>

TOTAL_BUY_DEBITS =
<importe>

TOTAL_SELL_GROSS =
<importe>

TOTAL_SELL_COMMISSIONS =
<importe>

NET_SELL_PROCEEDS =
<importe>

NET_BATCH_CASH_EFFECT =
<importe>

OPENING_BOOK_CASH =
<importe>

CLOSING_CALCULATED_BOOK_CASH =
<importe>

BROKER_CASH =
<importe/NOT_PROVIDED>

CASH_RECONCILIATION_DIFFERENCE =
<importe/UNKNOWN>

CASH_RECONCILIATION_STATUS =
PASS /
REQUIRED /
BROKER_CASH_NOT_PROVIDED

PANW_EXECUTED_QUANTITY =
0.33422822

PANW_EXECUTED_PRICE =
331.51

PANW_GROSS =
110.80

PANW_COMMISSION =
0.52

PANW_CASH_DEBIT =
111.32

PANW_REGISTERED =
PASS/FAIL

PANW_MOVEMENT_ID =
<id>

DECISIONS_WITHOUT_MOVEMENT_AFTER =
0

MOVEMENTS_WITHOUT_RECOMMENDATION_AFTER =
0

PARTIAL_WRITES_AFTER =
0

DUPLICATE_MOVEMENTS_AFTER =
0

HOLDINGS_RECONCILED =
PASS/FAIL

BOOK_RECONCILED =
PASS/FAIL

LEDGER_RECONCILED =
PASS/FAIL

CASH_RECONCILED =
PASS/FAIL/NOT_VERIFIABLE

NAV_RECALCULATED =
PASS/FAIL

NO_WIO_ORDERS_SENT =
PASS/FAIL

NO_DUPLICATE_EXECUTIONS =
PASS/FAIL

SECRET_SCAN =
PASS/FAIL

NO_PUSH_PERFORMED =
PASS/FAIL

LOCAL_COMMIT =
<hash>

EXACT_REMAINING_MANUAL_DATA:
1. <ticker y dato faltante>
2. <ticker y dato faltante>
3. <ticker y dato faltante>

Detenerse después del hotfix y la reconciliación del lote.