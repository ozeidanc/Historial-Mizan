MIZAN · CARTERA DE CRECIMIENTO
ORDEN DEFINITIVA
APORTACIONES REALES + RECÁLCULO AUTOMÁTICO
ELIMINACIÓN TOTAL DE “PROPUESTA DE RESIZING POR CAPITAL”

FECHA DE REFERENCIA =
2026-07-27

NATURALEZA

Esta orden sustituye todas las instrucciones anteriores relacionadas con:

- capital resizing;
- propuestas crp;
- aportaciones de capital;
- recalibración por cambio de capital;
- eliminación del panel de resizing.

Omar confirma que ya reintrodujo correctamente todos los fills reales de Wio
que estaban pendientes.

El trabajo debe continuar desde el estado real del repositorio, sin repetir
fills, sin crear movimientos duplicados y sin reabrir Gate 2F.

Objetivo final:

1. verificar y cerrar la reconciliación del lote Wio;
2. eliminar completamente el resizing abstracto;
3. impedir que vuelvan a generarse propuestas crp;
4. implementar “Registrar aportación de capital”;
5. registrar aportaciones reales en Libro Mayor y cash;
6. preservar neutralidad de rentabilidad;
7. recalcular automáticamente las recomendaciones;
8. respetar mercado cerrado, mercado abierto y cutoff preopen;
9. versionar y sustituir revisiones pendientes;
10. desplegar y validar el flujo completo;
11. no crear ninguna aportación productiva ficticia;
12. no enviar ninguna orden a Wio.

==================================================
0. BASELINE ESPERADO
==================================================

CANONICAL_HEAD esperado =
ce510c5

SCHEMA esperado =
v30

Si el HEAD o schema encontrados son diferentes:

- inventariar el estado;
- no resetear;
- no descartar cambios;
- identificar commits posteriores;
- continuar únicamente si el estado es coherente;
- reportar el baseline real.

Estado conocido:

HOTFIX_WIO_POST_TRADE =
DEPLOYED

PANW_MOVEMENT_ID =
91

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

CRP_AUDITED =
3/3

CRP_REAL_WIO_EXECUTIONS =
0

CRP_NON_ECONOMIC_DECISIONS =
3

CRP_REMOVED_FROM_WIO_REENTRY_QUEUE =
3

Omar confirma:

ALL_WIO_FILLS_REENTERED =
TRUE

Gate 2F permanece:

INACTIVE

La política prospectiva permanece:

APPROVED_PENDING_ACTIVATION

effective_from permanece:

NULL

No modificar ni activar Gate 2F durante esta orden.

==================================================
1. PRECONDICIONES
==================================================

Antes de modificar código o datos:

- confirmar HEAD;
- confirmar schema;
- confirmar una sola raíz Git;
- inventariar archivos modificados;
- inventariar archivos untracked;
- confirmar una sola instancia operativa;
- comprobar /ping;
- comprobar env-info;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- crear backup SQLite;
- calcular SHA-256;
- abrir y validar el backup;
- capturar baseline productivo completo.

Capturar:

- review runs;
- recomendaciones;
- decisiones;
- execution details;
- movimientos;
- cash events;
- holdings;
- comisiones;
- Book;
- cash;
- NAV;
- capital objetivo;
- aportaciones existentes;
- crp históricas;
- tablas prospectivas de Gate 2F;
- política prospectiva.

No continuar si:

- existen foreign keys rotas;
- integrity_check falla;
- existen dos instancias;
- el backup no abre;
- aparecen movimientos duplicados;
- los fills registrados no pueden vincularse a recomendaciones.

==================================================
2. VERIFICAR LOS FILLS WIO YA REGISTRADOS
==================================================

No asumir que siguen pendientes 18 fills.

Volver a consultar dinámicamente el estado después de la introducción realizada
por Omar.

Para cada recomendación ejecutada en Wio verificar:

- recommendation_id;
- ticker;
- action;
- movement_id;
- execution detail;
- executed_quantity;
- executed_price;
- commission;
- execution_date;
- cash events;
- holding delta;
- decision status;
- registration status;
- idempotency key.

Resultado esperado:

WIO_REENTRY_RECOMMENDATIONS_REMAINING =
0

No contar como fills:

- las tres decisiones crp no económicas;
- decisiones superseded;
- resizing abstracto;
- recomendaciones que nunca representaron una operación Wio.

No volver a registrar automáticamente ninguna operación.

Si alguna recomendación no tiene movement_id:

- identificarla;
- no inventar datos;
- no duplicar;
- comprobar si el movimiento existe mediante otra referencia;
- bloquear la reconciliación solo para el caso afectado.

==================================================
3. RECONCILIACIÓN DE MOVIMIENTOS
==================================================

Para cada compra:

expected_holding_delta =
+executed_quantity

expected_gross =
executed_quantity × executed_price

expected_cash_effect =
-(expected_gross + commission)

Para cada venta:

expected_holding_delta =
-executed_quantity

expected_gross =
executed_quantity × executed_price

expected_cash_effect =
expected_gross - commission

Verificar:

- cantidades reales preservadas;
- precios reales preservados;
- principal separado de comisión;
- cash events correctos;
- holding actualizado;
- movement vinculado;
- recommendation vinculada;
- decisión aceptada después del movimiento;
- cero estimaciones usadas como fill;
- cero duplicados.

Declarar:

DECISIONS_WITHOUT_MOVEMENT =
<número>

MOVEMENTS_WITHOUT_RECOMMENDATION =
<número>

PARTIAL_WRITES =
<número>

DUPLICATE_MOVEMENTS =
<número>

El resultado requerido para continuar es:

DECISIONS_WITHOUT_MOVEMENT = 0
MOVEMENTS_WITHOUT_RECOMMENDATION = 0
PARTIAL_WRITES = 0
DUPLICATE_MOVEMENTS = 0

Las tres crp no económicas quedan excluidas de estas invariantes operativas.

==================================================
4. RECONCILIACIÓN DE CASH
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

Comparar con:

- cash events;
- cash del Book;
- cash mostrado por Mizan;
- cash del broker, si Omar lo proporcionó.

Distinguir:

A. INTERNAL_BOOK_CASH_RECONCILED

La contabilidad interna reconcilia con sus movimientos y eventos.

B. BROKER_CASH_RECONCILED

El saldo interno reconcilia con el saldo disponible mostrado por Wio.

Si no se proporcionó cash final de Wio:

BROKER_CASH_RECONCILED =
NOT_VERIFIABLE

Esto no impide completar el desarrollo si:

- el ledger interno reconcilia;
- todos los fills están registrados;
- holdings reconcilian;
- no existen partial writes;
- no existen duplicados.

Pero debe mostrarse de forma honesta:

BROKER_CASH_VERIFICATION =
PENDING

No inventar ni inferir el cash de Wio.

==================================================
5. RECONCILIACIÓN DE HOLDINGS Y NAV
==================================================

Reconstruir los holdings finales desde:

- holdings iniciales;
- compras registradas;
- ventas registradas;
- eliminaciones registradas;
- cantidades reales.

Comparar con los holdings actuales del Book.

Recalcular NAV mediante:

NAV =
cash
+ market value de holdings
+ otros componentes canónicos existentes

No usar:

- cantidades recomendadas;
- importes estimados;
- capital objetivo como cash;
- crp no económicas.

Declarar:

BOOK_RECONCILED =
PASS/FAIL

LEDGER_RECONCILED =
PASS/FAIL

HOLDINGS_RECONCILED =
PASS/FAIL

COMMISSIONS_RECONCILED =
PASS/FAIL

INTERNAL_BOOK_CASH_RECONCILED =
PASS/FAIL

NAV_RECALCULATED =
PASS/FAIL

==================================================
6. CIERRE DEL INCIDENTE WIO
==================================================

Si quedan verdes:

- WIO_REENTRY_RECOMMENDATIONS_REMAINING = 0;
- DECISIONS_WITHOUT_MOVEMENT = 0;
- PARTIAL_WRITES = 0;
- DUPLICATE_MOVEMENTS = 0;
- BOOK_RECONCILED = PASS;
- LEDGER_RECONCILED = PASS;
- HOLDINGS_RECONCILED = PASS;
- INTERNAL_BOOK_CASH_RECONCILED = PASS;

establecer:

GROWTH_BOOK_RECONCILIATION_STATUS =
RECONCILED

Si falla una invariante:

GROWTH_BOOK_RECONCILIATION_STATUS =
BLOCKED

y no habilitar la confirmación productiva de aportaciones.

No volver a solicitar los 18 fills si ya están correctamente registrados.

==================================================
7. ELIMINAR COMPLETAMENTE EL RESIZING ABSTRACTO
==================================================

Eliminar de la operación normal:

- panel “Propuesta de resizing por capital”;
- botones Aceptar/Declinar asociados a crp;
- generación de capital_resizing_line;
- contadores operativos de crp;
- crp dentro de recomendaciones normales;
- mensajes de Libro Mayor para decisiones crp;
- crp dentro de la cola Wio;
- crp dentro de acciones pendientes;
- cualquier endpoint que cree nuevas propuestas de resizing abstracto.

Resultado requerido:

GENERIC_CAPITAL_RESIZING_PROPOSAL_REMOVED =
PASS

NEW_CRP_RECOMMENDATIONS_POSSIBLE =
NO

No borrar las tres decisiones históricas.

Conservarlas como:

LEGACY_NON_ECONOMIC_CAPITAL_RESIZING_DECISION

Deben seguir disponibles únicamente para auditoría.

No deben:

- crear movimientos;
- modificar holdings;
- modificar cash;
- modificar NAV;
- solicitar fills;
- aparecer como operaciones pendientes;
- aparecer como registradas en Libro Mayor.

==================================================
8. DISTINGUIR TRES CONCEPTOS
==================================================

A. CAPITAL OBJETIVO

Es una configuración.

Cambiar:

50.000
→
60.000

no implica que existan 10.000 adicionales en Wio.

Por sí solo:

- no crea cash;
- no aumenta NAV;
- no crea movimiento;
- no genera recomendaciones financiadas;
- no activa recalibración económica.

Resultado:

CAPITAL_TARGET_CHANGE_CREATES_CASH =
NO

B. APORTACIÓN REAL

Es una entrada externa de dinero ya disponible en Wio.

Debe:

- registrarse en Libro Mayor;
- aumentar cash;
- aumentar NAV;
- conservar rentabilidad;
- programar una recalibración.

C. RECOMENDACIONES

Son el resultado posterior del motor.

No son aportaciones.

No crean dinero.

No sustituyen el registro económico de la aportación.

==================================================
9. FLUJO DE USUARIO DEFINITIVO
==================================================

El flujo operativo debe ser:

1. Omar aporta dinero en Wio.
2. Espera a que aparezca disponible.
3. Abre Mizan.
4. Pulsa “Registrar aportación de capital”.
5. Introduce el importe real.
6. Confirma que ya está disponible en Wio.
7. Mizan registra el flujo.
8. Cash y NAV aumentan.
9. Mizan programa el recálculo.
10. Se genera una revisión nueva.
11. Omar ejecuta las recomendaciones en Wio.
12. Omar registra los fills reales en Mizan.

Mizan no envía órdenes a Wio.

==================================================
10. REGISTRAR APORTACIÓN DE CAPITAL
==================================================

Crear en la cartera de crecimiento:

REGISTRAR APORTACIÓN DE CAPITAL

Campos:

- account_id o portfolio_id;
- contribution_amount;
- currency;
- available_date;
- available_time nullable;
- Wio reference nullable;
- notes nullable;
- idempotency key;
- confirmed_by;
- confirmed_at.

Confirmación explícita:

“Confirmo que este importe ya está disponible para operar en Wio.”

Estados:

- DRAFT;
- PENDING_AVAILABILITY;
- CONFIRMED_AVAILABLE;
- CANCELLED;
- REJECTED.

Solo:

CONFIRMED_AVAILABLE

puede modificar el Libro Mayor y cash.

No utilizar:

- DRAFT;
- intención;
- transferencia pendiente;
- capital objetivo;
- importe estimado;

como dinero disponible.

==================================================
11. PREVIEW DE APORTACIÓN
==================================================

Antes de confirmar mostrar:

- cartera;
- importe;
- moneda;
- fecha de disponibilidad;
- cash actual;
- cash proyectado;
- NAV actual;
- NAV proyectado;
- estado de reconciliación;
- mercado USA abierto/cerrado;
- próxima sesión;
- sesión objetivo del recálculo;
- revisión vigente afectada;
- advertencias;
- contribution preview hash.

El preview:

- no escribe;
- no aumenta cash;
- no aumenta NAV;
- no crea revisión;
- no supersede recomendaciones.

Si el Book no está reconciliado:

BOOK_RECONCILIATION_REQUIRED

y no permitir confirmación.

==================================================
12. REGISTRO ECONÓMICO ATÓMICO
==================================================

Confirmar una aportación debe realizarse en una única transacción:

1. cargar preview;
2. validar reconciliación;
3. validar importe;
4. validar moneda;
5. validar disponibilidad;
6. comprobar idempotencia;
7. detectar duplicados;
8. crear contribution detail;
9. crear movimiento en Libro Mayor;
10. crear cash event;
11. aumentar cash;
12. aumentar NAV;
13. registrar neutralidad de flujo;
14. crear recalculation request;
15. registrar audit event;
16. COMMIT.

Tipo de movimiento:

EXTERNAL_CAPITAL_CONTRIBUTION

La aportación no crea:

- BUY;
- SELL;
- ticker;
- holding;
- comisión;
- orden;
- fill;
- P&L.

Si falla:

- ROLLBACK completo;
- cash no cambia;
- NAV no cambia;
- no se crea recálculo;
- no se muestra éxito.

==================================================
13. EFECTO ECONÓMICO
==================================================

Debe cumplirse:

cash_after =
cash_before + contribution_amount

NAV_after =
NAV_before + contribution_amount

holding_quantities_after =
holding_quantities_before

commission =
0

investment_return_created =
0

La aportación es un flujo externo.

No debe mostrarse como rentabilidad.

Si existe metodología de unidades:

units_issued =
contribution_amount / unit_value_before_flow

unit_value_after_flow =
unit_value_before_flow

Si la cartera real utiliza otro mecanismo para neutralizar flujos:

- emplear el mecanismo canónico;
- documentarlo;
- probarlo.

==================================================
14. SCHEMA
==================================================

Auditar el soporte existente para:

- contribution detail;
- contribution status;
- availability timestamp;
- idempotency;
- ledger movement;
- recalculation request;
- review supersession;
- contribution-review lineage;
- applicable session;
- cutoff;
- input hash.

Si v30 ya soporta correctamente el contrato:

- no migrar.

Si falta soporte:

- usar NEXT_AVAILABLE_SCHEMA_VERSION;
- crear migración aditiva mínima;
- no reutilizar crp;
- no cambiar la semántica de tablas históricas;
- probar migración;
- probar rollback seguro o contrato forward-only;
- migrar Producción solo después de preview verde.

No introducir una aportación ficticia durante la migración.

==================================================
15. GATE DE RECONCILIACIÓN
==================================================

Crear:

isGrowthCapitalContributionAllowed

Debe devolver true solo si:

- Book reconciliado;
- Libro Mayor reconciliado;
- holdings reconciliados;
- cash interno reconciliado;
- cero fills Wio pendientes de registro;
- cero partial writes;
- cero duplicados;
- una sola instancia;
- schema compatible.

Crear:

isGrowthCapitalRecalculationAllowed

Debe exigir además:

- aportación CONFIRMED_AVAILABLE;
- precios canónicos disponibles;
- holdings snapshot sellado;
- sesión objetivo resuelta;
- revisión vigente identificada;
- input hash válido.

No hardcodear el número 18.

Calcular dinámicamente cualquier fill pendiente.

==================================================
16. REGLA TEMPORAL · MERCADO CERRADO
==================================================

Usar:

America/New_York

y el calendario bursátil canónico estadounidense.

Si la aportación se confirma con el mercado regular cerrado:

- registrar inmediatamente cash y NAV;
- determinar la próxima sesión;
- invalidar operativamente recomendaciones pendientes no ejecutadas;
- programar una revisión nueva;
- calcularla antes de la próxima apertura, sujeto al cutoff.

Incluye:

- after-hours;
- overnight;
- premarket;
- fin de semana;
- festivo.

Estado:

RECALCULATION_SCHEDULED_FOR_NEXT_OPEN

==================================================
17. REGLA TEMPORAL · MERCADO ABIERTO
==================================================

Si la aportación se confirma durante la sesión regular:

- registrar inmediatamente cash y NAV;
- no recalibrar operativamente a mitad de sesión;
- no modificar recomendaciones ejecutadas;
- no modificar fills pendientes de registro;
- no publicar otra revisión operativa durante esa sesión;
- programar el recálculo antes de la siguiente apertura.

Estado:

RECALCULATION_SCHEDULED_FOR_NEXT_SESSION_PREOPEN

==================================================
18. CUTOFF PREOPEN
==================================================

Definir o reutilizar un cutoff configurable.

Si la aportación se confirma demasiado cerca de la apertura para:

- sellar holdings;
- reconciliar cash;
- resolver precios;
- ejecutar el motor;
- validar resultados;
- publicar con seguridad;

no publicar una revisión incompleta.

Estado:

RECALCULATION_DEFERRED_BY_CUTOFF

Resolver la siguiente ventana segura.

No hardcodear:

- 09:30;
- early close;
- festivos;
- fines de semana;

sin utilizar el calendario canónico.

==================================================
19. VERSIONADO DE REVISIONES
==================================================

Persistir en cada revisión:

- review_run_id;
- capital_basis;
- cash_basis;
- NAV_basis;
- holdings_snapshot_id;
- contribution_ids;
- applicable_market_session;
- generated_at;
- price_basis;
- price_session;
- input_hash;
- supersedes_review_run_id nullable;
- superseded_by_review_run_id nullable;
- supersession_reason nullable.

Motivo:

CAPITAL_CONTRIBUTION_CONFIRMED

No borrar ni sobrescribir la revisión anterior.

==================================================
20. TRATAMIENTO DE RECOMENDACIONES ANTERIORES
==================================================

A. PENDING_NOT_EXECUTED

- marcar SUPERSEDED_BY_CAPITAL_CHANGE;
- impedir aceptación como revisión vigente;
- generar reemplazo.

B. FORM_OPEN_NO_WIO_EXECUTION

- marcar formulario como obsoleto;
- conservar sus datos;
- impedir que se use contra la revisión nueva;
- ofrecer abrir la nueva recomendación.

C. EXECUTED_IN_WIO_PENDING_REGISTRATION

- no modificar;
- no invalidar el fill;
- registrar exactamente sus datos reales;
- conservar lineage a la revisión original.

D. ACCEPTED_AND_RECORDED

- no modificar;
- no revertir;
- conservar movimiento.

E. DECLINED

- conservar historia;
- permitir una nueva recomendación si el cálculo la justifica.

Una aportación no puede modificar retroactivamente una operación Wio.

==================================================
21. RECÁLCULO DESDE CERO
==================================================

Crear:

recalculateGrowthRecommendationsAfterCapitalContribution

Entrada:

- account/portfolio;
- contribution IDs;
- contribution total;
- reconciled cash;
- NAV;
- holdings snapshot;
- target allocation policy;
- certified price basis;
- applicable market session.

Debe:

1. validar el gate;
2. comprobar idempotencia;
3. resolver sesión;
4. cargar holdings reales;
5. cargar cash real;
6. cargar NAV;
7. cargar aportaciones;
8. cargar precios;
9. localizar revisión vigente;
10. clasificar recomendaciones;
11. preservar operaciones ejecutadas;
12. superseder pendientes;
13. recalcular la cartera completa;
14. crear review run nuevo;
15. crear recomendaciones normales;
16. vincular revisiones;
17. publicar cuando sea seguro;
18. registrar audit event.

No enviar órdenes.

==================================================
22. RECOMENDACIONES RESULTANTES
==================================================

El motor debe generar únicamente:

- MANTENER;
- INCORPORAR;
- AUMENTAR;
- REDUCIR;
- ELIMINAR.

No generar:

- CAPITAL_RESIZING;
- RESIZING_PROPOSAL;
- capital_resizing_line;
- recomendación genérica por importe aportado.

Para cada ticker:

recommended_trade_quantity =
target_quantity_at_updated_capital
-
current_actual_quantity

Recalcular desde cero utilizando:

- capital actualizado;
- cash disponible;
- NAV;
- holdings reales;
- política;
- selección;
- sizing;
- precios sellados.

No utilizar:

old_recommended_quantity
+ proportional_contribution_adjustment

==================================================
23. CAMBIO DE CAPITAL OBJETIVO SIN APORTACIÓN
==================================================

Si Omar cambia el capital objetivo pero no registra una aportación:

- guardar la configuración, si sigue siendo necesaria;
- no crear cash;
- no aumentar NAV;
- no crear movimiento;
- no ejecutar recálculo financiado;
- no generar compras con dinero inexistente.

Mostrar:

“El capital objetivo cambió, pero no existe una aportación confirmada y
disponible. El cash económico no ha cambiado.”

Si ya no existe un uso válido para capital objetivo:

- documentar su función;
- o retirarlo en un cambio separado;
- no confundirlo con aportaciones.

==================================================
24. UI DEFINITIVA
==================================================

Eliminar completamente:

PROPUESTA DE RESIZING POR CAPITAL

Añadir:

REGISTRAR APORTACIÓN DE CAPITAL

Mostrar estados:

- BORRADOR;
- PENDIENTE DE DISPONIBILIDAD;
- APORTACIÓN DISPONIBLE;
- RECÁLCULO PROGRAMADO;
- RECÁLCULO EN CURSO;
- NUEVA REVISIÓN DISPONIBLE;
- RECÁLCULO DIFERIDO POR CUTOFF;
- CONCILIACIÓN REQUERIDA;
- ERROR.

Después de registrar:

- importe;
- moneda;
- fecha de disponibilidad;
- movement_id;
- cash anterior;
- cash posterior;
- NAV anterior;
- NAV posterior;
- mercado abierto/cerrado;
- sesión objetivo;
- estado del recálculo.

En revisión anterior:

“Revisión sustituida por aportación de capital.”

En revisión nueva:

- capital anterior;
- aportación;
- capital actualizado;
- cash disponible;
- NAV;
- revisión sustituida;
- sesión aplicable;
- precios utilizados;
- fecha de generación.

==================================================
25. ENDPOINTS
==================================================

Crear o adaptar endpoints coherentes con el stack existente:

- preview contribution;
- create draft;
- confirm available contribution;
- cancel draft/pending contribution;
- get contribution;
- list contributions;
- get recalculation status;
- retry safe recalculation.

No crear endpoint de envío de órdenes.

Todos los endpoints de escritura deben:

- validar idempotencia;
- validar reconciliación;
- usar transacciones;
- devolver movement_id;
- devolver contribution_id;
- devolver recalculation status;
- conservar audit trail.

==================================================
26. SCHEDULER Y CATCH-UP
==================================================

Integrar el recálculo con el scheduler existente.

No crear un scheduler paralelo.

Debe procesar:

- recalculation requests pendientes;
- próxima apertura;
- siguiente sesión;
- cutoff;
- reinicio;
- servidor apagado;
- error recuperable;
- precios pendientes.

Startup catch-up:

- localiza requests pendientes;
- procesa cronológicamente;
- no duplica revisiones;
- no cambia aportaciones;
- no envía órdenes.

Una aportación confirmada no debe perder su recálculo por reinicio.

==================================================
27. CONCURRENCIA E IDEMPOTENCIA
==================================================

Probar:

- doble clic en confirmar;
- mismo broker reference;
- misma idempotency key;
- payload diferente con mismo key;
- dos aportaciones independientes;
- dos aportaciones antes del recálculo;
- dos workers;
- scheduler y catch-up;
- aportación durante generación;
- cambio de sesión;
- formulario antiguo abierto;
- reinicio antes del COMMIT;
- reinicio después del COMMIT.

Mismo aporte + mismo payload:

ALREADY_RECORDED

Mismo key + payload diferente:

IDEMPOTENCY_CONFLICT

Un mismo input económico:

- una sola revisión canónica;
- cero recomendaciones duplicadas.

==================================================
28. PRUEBAS
==================================================

Crear:

verify-growth-wio-batch-final-reconciliation.mjs
verify-growth-remove-capital-resizing.mjs
verify-growth-no-new-crp.mjs
verify-growth-capital-contribution-preview.mjs
verify-growth-capital-contribution-accounting.mjs
verify-growth-capital-contribution-neutrality.mjs
verify-growth-capital-target-no-cash.mjs
verify-growth-capital-contribution-market-closed.mjs
verify-growth-capital-contribution-market-open.mjs
verify-growth-capital-contribution-cutoff.mjs
verify-growth-capital-review-supersession.mjs
verify-growth-capital-executed-fill-preservation.mjs
verify-growth-capital-recalculation.mjs
verify-growth-capital-contribution-idempotency.mjs
verify-growth-capital-contribution-scheduler.mjs
verify-growth-capital-contribution-catchup.mjs
verify-growth-capital-contribution-ui.mjs

Casos obligatorios:

A. Todos los fills Wio registrados.
B. Tres crp excluidas correctamente.
C. Cero decisiones operativas sin movimiento.
D. Cero partial writes.
E. Cero duplicados.
F. Holdings reconciliados.
G. Cash interno reconciliado.
H. Panel de resizing eliminado.
I. No se crean nuevas crp.
J. Cambio de capital objetivo no crea cash.
K. Preview no escribe.
L. Aportación crea movimiento.
M. Aportación aumenta cash.
N. Aportación aumenta NAV.
O. Aportación no modifica holdings.
P. Aportación no crea rentabilidad.
Q. Mercado cerrado recalcula para próxima apertura.
R. Mercado abierto recalcula antes de la siguiente sesión.
S. Cutoff difiere de forma segura.
T. Revisión anterior queda versionada.
U. Pendientes quedan superseded.
V. Fill Wio ejecutado permanece intacto.
W. Movimiento registrado permanece intacto.
X. Recomendación declinada se conserva.
Y. Recálculo usa capital y cash actualizados.
Z. Recálculo se hace desde cero.
AA. Solo se generan recomendaciones normales.
AB. Doble aportación no duplica.
AC. Doble recálculo no duplica.
AD. Catch-up recupera el trabajo.
AE. No se envían órdenes a Wio.
AF. Gate 2F permanece inactivo.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- hotfix Wio;
- growth-core-end-to-end;
- capital-resizing-phase2;
- recommendation engine;
- Book;
- Libro Mayor;
- holdings;
- cash;
- NAV;
- calendar;
- scheduler;
- startup catch-up;
- repository integrity.

Corregir los seis fallos preexistentes de capital-resizing-phase2 mediante
aislamiento adecuado de fixtures.

No eliminar aserciones.
No añadir skips silenciosos.
No modificar Producción para satisfacer tests.

==================================================
29. CHROME
==================================================

Validar en Chrome:

A. Los fills registrados muestran movement_id.
B. No queda CASH_OVERSPEND_BLOCKED.
C. No quedan fills pendientes.
D. El panel de resizing no existe.
E. Las tres crp no aparecen operativamente.
F. Existe “Registrar aportación de capital”.
G. El preview muestra cash/NAV proyectados.
H. Cambiar capital objetivo no crea cash.
I. Un fixture o entorno seguro demuestra confirmación de aporte.
J. Mercado cerrado muestra próxima apertura.
K. Mercado abierto muestra siguiente sesión.
L. Revisión anterior muestra supersession.
M. Revisión nueva muestra capital/cash nuevos.
N. No hay errores JavaScript.
O. No se envían órdenes a Wio.

No crear una aportación ficticia en Producción para validar Chrome.

Utilizar:

- fixture;
- copia;
- entorno LAB;
- modo de prueba aislado.

==================================================
30. PREVIEW PRODUCTIVO
==================================================

Antes de desplegar:

- backup;
- SHA-256;
- integrity_check;
- foreign_key_check;
- baseline;
- ejecutar migración en copia, si existe;
- iniciar servidor sobre copia;
- ejecutar suites;
- ejecutar Chrome;
- comparar Book;
- comparar holdings;
- comparar cash;
- comparar NAV;
- verificar crp;
- verificar que no se genera revisión sin aportación.

No desplegar si:

- los fills no reconcilian;
- el panel sigue generando crp;
- una aportación crea rentabilidad;
- capital objetivo crea cash;
- se modifican holdings por aportar;
- se duplica una revisión;
- aparece una orden al broker.

==================================================
31. DESPLIEGUE
==================================================

Orden:

1. detener escrituras;
2. una sola instancia;
3. backup productivo;
4. SHA-256;
5. integrity_check;
6. foreign_key_check;
7. baseline;
8. aplicar migración, si corresponde;
9. desplegar backend;
10. desplegar UI;
11. reiniciar una sola instancia;
12. /ping;
13. env-info;
14. validar Book;
15. validar holdings;
16. validar cash;
17. validar NAV;
18. validar panel eliminado;
19. validar flujo de aportación;
20. validar scheduler;
21. validar catch-up;
22. verificar cero aportaciones productivas nuevas;
23. verificar cero revisiones productivas nuevas;
24. verificar cero órdenes Wio.

El despliegue habilita la funcionalidad.

No crea una aportación automáticamente.

No genera una revisión automáticamente sin una aportación real.

==================================================
32. SEGURIDAD PRODUCTIVA
==================================================

Después del despliegue:

PRODUCTION_CONTRIBUTIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_REVIEWS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_RECOMMENDATIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_BROKER_ORDERS_SENT =
0

Las próximas acciones económicas solo pueden comenzar cuando Omar:

- registre una aportación;
- confirme que está disponible en Wio.

==================================================
33. GATE 2F
==================================================

Gate 2F permanece aparcado.

No modificar:

- status de política;
- effective_from;
- configuración prospectiva;
- reservas prospectivas;
- endpoints prospectivos;
- UI prospectiva de La Lente.

Resultado:

GATE_2F_REMAINS_INACTIVE =
PASS

==================================================
34. COMMIT
==================================================

Crear un único commit local para esta modificación:

fix(growth): replace capital resizing with contribution recalculation

Si el trabajo requiere una migración y el repositorio exige commits separados:

- preferir aun así un único commit coherente;
- explicar cualquier excepción.

Ejecutar secret scan.

No crear tag.
No hacer push.

==================================================
35. ENTREGA FINAL
==================================================

CANONICAL_HEAD_BEFORE =
<hash real, esperado ce510c5>

CANONICAL_HEAD_AFTER =
<hash>

SCHEMA_BEFORE =
v30

SCHEMA_AFTER =
<v30/NEXT_AVAILABLE_SCHEMA_VERSION>

WIO_REENTRY_RECOMMENDATIONS_REMAINING =
<número>

WIO_FILLS_REGISTERED_TOTAL =
<número>

DECISIONS_WITHOUT_MOVEMENT =
<número>

PARTIAL_WRITES =
<número>

DUPLICATE_MOVEMENTS =
<número>

BOOK_RECONCILED =
PASS/FAIL

LEDGER_RECONCILED =
PASS/FAIL

HOLDINGS_RECONCILED =
PASS/FAIL

COMMISSIONS_RECONCILED =
PASS/FAIL

INTERNAL_BOOK_CASH_RECONCILED =
PASS/FAIL

BROKER_CASH_RECONCILED =
PASS/FAIL/NOT_VERIFIABLE

NAV_RECALCULATED =
PASS/FAIL

GROWTH_BOOK_RECONCILIATION_STATUS =
RECONCILED/BLOCKED

GENERIC_CAPITAL_RESIZING_PROPOSAL_REMOVED =
PASS/FAIL

LEGACY_CRP_DECISIONS_PRESERVED =
3/3

CRP_REAL_WIO_EXECUTIONS =
0

NEW_CRP_RECOMMENDATIONS_POSSIBLE =
NO/YES

CAPITAL_TARGET_CHANGE_CREATES_CASH =
NO

CAPITAL_CONTRIBUTION_FLOW =
PASS/FAIL

CAPITAL_CONTRIBUTION_PREVIEW =
PASS/FAIL

CAPITAL_CONTRIBUTION_LEDGER_MOVEMENT =
PASS/FAIL

CONFIRMED_CONTRIBUTION_INCREASES_CASH =
PASS/FAIL

CONFIRMED_CONTRIBUTION_INCREASES_NAV =
PASS/FAIL

CAPITAL_CONTRIBUTION_RETURN_NEUTRAL =
PASS/FAIL

CAPITAL_CONTRIBUTION_HOLDINGS_UNCHANGED =
PASS/FAIL

MARKET_CLOSED_RECALCULATION =
PASS/FAIL

MARKET_OPEN_NEXT_SESSION_RECALCULATION =
PASS/FAIL

PREOPEN_CUTOFF =
PASS/FAIL

OLD_REVIEW_VERSIONED =
PASS/FAIL

PENDING_RECOMMENDATIONS_SUPERSEDED =
<número en fixtures/pruebas>

EXECUTED_WIO_FILLS_PRESERVED =
PASS/FAIL

RECORDED_RECOMMENDATIONS_PRESERVED =
PASS/FAIL

RECALCULATION_FROM_FULL_CANONICAL_STATE =
PASS/FAIL

GENERIC_RESIZING_RECOMMENDATIONS_CREATED =
0

CONTRIBUTION_IDEMPOTENCY =
PASS/FAIL

RECALCULATION_IDEMPOTENCY =
PASS/FAIL

SCHEDULER_INTEGRATION =
PASS/FAIL

STARTUP_CATCHUP =
PASS/FAIL

CHROME_GREEN =
PASS/FAIL

JAVASCRIPT_ERRORS =
<número>

PRODUCTION_CONTRIBUTIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_REVIEWS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_BROKER_ORDERS_SENT =
0

GATE_2F_REMAINS_INACTIVE =
PASS/FAIL

SECRET_SCAN =
PASS/FAIL

NO_PUSH_PERFORMED =
PASS/FAIL

NO_TAG_CREATED =
PASS/FAIL

LOCAL_COMMIT =
<hash>

EXACT_REMAINING_BLOCKERS:
1. <bloqueo o NONE>
2. <bloqueo o NONE>
3. <bloqueo o NONE>

Detenerse después de completar, desplegar y verificar la modificación.
No crear una aportación productiva de prueba.