MIZAN · CARTERA DE CRECIMIENTO
APORTACIÓN DE CAPITAL Y RECALIBRACIÓN AUTOMÁTICA
ELIMINACIÓN DE “PROPUESTA DE RESIZING POR CAPITAL”

PRIORIDAD

Implementar después de cerrar la introducción y reconciliación de los fills
Wio pendientes.

Gate 2F continúa aparcado.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado =
b9bfa01

SCHEMA esperado =
v30

Hotfix Wio desplegado:

POST_TRADE_REGISTRATION_MODE =
ACTIVE

CASH_OVERSPEND_BLOCKS_ON_POST_TRADE_REGISTRATION =
0

PANW movement_id =
91

Todavía están pendientes de introducción manual:

- 18 fills de session-6;
- 3 accepted orphaned de crp;
- validación Chrome A–H;
- conciliación final de cash.

No generar una revisión real recalibrada mientras:

- los fills pendientes no estén registrados;
- holdings no estén reconciliados;
- cash no esté reconciliado o explícitamente marcado como base aceptada por
  Omar.

No utilizar un estado económico incompleto como base silenciosa.

==================================================
1. PROBLEMA FUNCIONAL
==================================================

Actualmente existe un panel o flujo denominado:

PROPUESTA DE RESIZING POR CAPITAL

Se han identificado al menos tres decisiones huérfanas crp:

- AAPL;
- ABNB;
- AAPL.

El panel permite aceptar una decisión, pero la aceptación puede no implicar:

- movimiento de Libro Mayor;
- aportación externa;
- aumento de cash;
- modificación del NAV;
- modificación económica de la cartera.

Este comportamiento no tiene sentido operativo.

Una decisión abstracta de “resizing por capital” no debe generar
recomendaciones ni aparecer como registrada si no existe una aportación
económica real.

Eliminar la propuesta de resizing como familia de recomendaciones.

==================================================
2. CONTRATO CORRECTO
==================================================

El flujo correcto debe ser:

OMAR APORTA CAPITAL EN WIO
→ CONFIRMA EN MIZAN QUE EL DINERO ESTÁ DISPONIBLE
→ MIZAN REGISTRA LA APORTACIÓN EN EL LIBRO MAYOR
→ AUMENTA CASH
→ AUMENTA NAV
→ CONSERVA EL P&L
→ INVALIDA RECOMENDACIONES PENDIENTES OBSOLETAS
→ RECALCULA LA CARTERA CON EL NUEVO CAPITAL
→ GENERA RECOMENDACIONES NORMALES POR TICKER

Las recomendaciones resultantes deben ser:

- MANTENER;
- INCORPORAR;
- AUMENTAR;
- REDUCIR;
- ELIMINAR.

No debe existir una recomendación independiente:

RESIZING POR CAPITAL

La aportación modifica automáticamente la base económica de la siguiente
revisión.

==================================================
3. DIFERENCIA ENTRE CAPITAL OBJETIVO Y APORTACIÓN
==================================================

Distinguir estrictamente:

A. CAPITAL OBJETIVO

Es una preferencia o configuración.

Cambiarlo por sí solo:

- no crea cash;
- no crea NAV;
- no crea movimiento;
- no prueba que el dinero exista en Wio;
- no autoriza recomendaciones financiadas.

B. APORTACIÓN CONFIRMADA

Es un flujo económico externo real.

Solo puede considerarse disponible cuando Omar confirma que:

- el dinero llegó a Wio;
- está disponible para operar;
- el importe es exacto;
- la fecha es exacta;
- la moneda es correcta.

Solo entonces:

cash += contribution_amount

NAV += contribution_amount

La aplicación no puede asumir:

capital objetivo nuevo
− capital anterior
=
cash disponible.

==================================================
4. ELIMINAR EL PANEL DE RESIZING
==================================================

Eliminar de la UI productiva:

- panel “Propuesta de resizing por capital”;
- botones Aceptar/Declinar asociados al resizing abstracto;
- mensajes “Registrada en el Book” basados en decisiones crp;
- generación futura de recomendaciones crp;
- aceptación de resizing sin flujo económico.

No eliminar historia existente.

Las tres decisiones crp actuales deben conservarse como auditoría legacy,
pero clasificarse como:

LEGACY_NON_ECONOMIC_CAPITAL_RESIZING_DECISION

No deben:

- crear movimientos;
- modificar holdings;
- modificar cash;
- contar como fills Wio pendientes;
- mostrarse como operaciones ejecutadas;
- mostrarse como “Registradas en el Libro Mayor”.

Eliminar esas tres crp de la lista de fills Wio pendientes si se confirma que
no correspondieron a operaciones reales ejecutadas en Wio.

Entregar:

LEGACY_CRP_DECISIONS =
<número>

LEGACY_CRP_ECONOMIC_MOVEMENTS =
0

LEGACY_CRP_HIDDEN_FROM_OPERATIONAL_QUEUE =
PASS/FAIL

Importante:

No confundir una recomendación normal de AAPL o ABNB que sí fue ejecutada en
Wio con una decisión crp no económica del mismo ticker.

Resolver mediante IDs y lineage.

==================================================
5. REGISTRO DE APORTACIÓN
==================================================

Crear o reutilizar un flujo explícito:

REGISTRAR APORTACIÓN DE CAPITAL

Campos obligatorios:

- cuenta/cartera;
- importe;
- moneda;
- fecha en que quedó disponible;
- hora opcional;
- referencia Wio opcional;
- comentario opcional;
- idempotency key.

Confirmación explícita:

“Confirmo que este importe ya está disponible en Wio.”

Estados:

- DRAFT;
- PENDING_AVAILABILITY;
- CONFIRMED_AVAILABLE;
- CANCELLED;
- REJECTED.

Solo:

CONFIRMED_AVAILABLE

puede modificar cash y activar una recalibración.

No utilizar aportaciones:

- prometidas;
- pendientes de transferencia;
- rechazadas;
- canceladas;
- estimadas.

==================================================
6. REGISTRO ECONÓMICO DE LA APORTACIÓN
==================================================

La confirmación debe registrar atómicamente:

1. contribution detail;
2. movimiento de Libro Mayor;
3. evento de cash;
4. aumento de NAV;
5. vínculo con la cuenta/cartera;
6. audit event;
7. solicitud de recalibración;
8. COMMIT.

Fórmulas:

cash_after =
cash_before + contribution_amount

NAV_after =
NAV_before + contribution_amount

La aportación no debe:

- modificar holdings;
- crear una compra;
- crear una venta;
- generar comisión;
- modificar precios de entrada;
- crear P&L;
- crear rentabilidad;
- enviar órdenes a Wio.

Debe registrarse como:

EXTERNAL_CAPITAL_CONTRIBUTION

No como:

BUY
SELL
RESIZING_RECOMMENDATION

==================================================
7. NEUTRALIDAD DE RENTABILIDAD
==================================================

La aportación es un flujo externo.

Debe cumplirse:

investment_return_created_by_contribution =
0

Si el sistema utiliza unidades:

units_issued =
contribution_amount / unit_value_before_flow

unit_value_after_flow =
unit_value_before_flow

Si la cartera real no utiliza un ledger de unidades, aplicar la metodología
canónica equivalente para excluir el flujo del rendimiento.

No mostrar una ganancia por el mero aumento de NAV.

==================================================
8. FUENTE DEL CASH PARA LA REVISIÓN
==================================================

La nueva revisión debe utilizar:

reconciled_cash
+ confirmed_available_contributions
- committed_valid_cash
- otros débitos económicos confirmados

No utilizar:

- capital objetivo;
- resizing aceptado;
- aportación pendiente;
- fills Wio todavía no registrados;
- ventas recomendadas no ejecutadas;
- proceeds estimados;
- cash calculado desde recomendaciones.

Antes de recalcular:

- registrar los fills Wio conocidos;
- actualizar holdings;
- contabilizar compras, ventas y comisiones;
- determinar cash;
- añadir la aportación confirmada;
- sellar la base económica.

Persistir:

- opening cash;
- contribution amount;
- post-contribution cash;
- holdings snapshot;
- NAV;
- reconciliation status;
- input hash.

==================================================
9. REGLA TEMPORAL · MERCADO USA CERRADO
==================================================

Usar:

America/New_York

y el calendario bursátil canónico de Estados Unidos.

Si la aportación queda CONFIRMED_AVAILABLE cuando la sesión regular está
cerrada:

- registrar inmediatamente la aportación;
- aumentar cash y NAV;
- localizar la revisión vigente;
- invalidar recomendaciones pendientes no ejecutadas;
- recalcular con el nuevo capital;
- publicar una nueva revisión antes de la próxima apertura, si existe tiempo
  operativo suficiente.

Esto incluye:

- después del cierre;
- overnight;
- premarket;
- fines de semana;
- festivos.

Si se confirma antes de la apertura regular y antes del cutoff operativo:

RECALCULATION_TARGET =
NEXT_MARKET_OPEN

La nueva revisión debe estar disponible antes de la próxima apertura.

==================================================
10. REGLA TEMPORAL · MERCADO USA ABIERTO
==================================================

Si la aportación queda CONFIRMED_AVAILABLE durante la sesión regular:

- registrar inmediatamente la aportación;
- aumentar cash y NAV;
- no recalibrar operativamente a mitad de sesión;
- no modificar recomendaciones ya ejecutadas;
- no modificar fills Wio pendientes de registro;
- no publicar una segunda revisión operativa durante la sesión;
- programar la recalibración para antes de la siguiente apertura regular.

Resultado:

RECALCULATION_TARGET =
NEXT_TRADING_SESSION_PREOPEN

La revisión recalculada debe utilizar:

- holdings reales después de todos los fills registrados;
- cash confirmado;
- aportación;
- base de precios canónica;
- políticas vigentes.

==================================================
11. CUTOFF PREOPEN
==================================================

Definir un cutoff configurable antes de la apertura.

Si la aportación se confirma demasiado cerca de la apertura para:

- reconciliar cash;
- sellar holdings;
- obtener precios;
- ejecutar el motor;
- validar resultados;

no publicar una revisión incompleta.

Estado:

RECALCULATION_DEFERRED_BY_CUTOFF

Programar para:

NEXT_TRADING_SESSION_PREOPEN

No hardcodear el horario sin utilizar:

- calendario canónico;
- zona America/New_York;
- configuración operativa;
- early closes;
- festivos.

==================================================
12. RECOMENDACIONES EXISTENTES
==================================================

Cuando una aportación cambia la base económica, clasificar la revisión
anterior.

A. PENDING_NOT_EXECUTED

- marcar SUPERSEDED_BY_CAPITAL_CHANGE;
- impedir que se ejecute como recomendación vigente;
- generar una recomendación nueva.

B. FORM_OPEN_NO_WIO_EXECUTION

- marcar como obsoleta;
- conservar datos introducidos;
- impedir que se registre como fill de la revisión nueva;
- ofrecer abrir la recomendación recalculada.

C. EXECUTED_IN_WIO_PENDING_REGISTRATION

- no invalidar;
- no cambiar cantidad;
- no cambiar precio;
- no cambiar comisión;
- registrar exactamente el fill real;
- conservar lineage a la revisión original.

D. ACCEPTED_AND_RECORDED

- no modificar;
- no revertir;
- no recalcular retrospectivamente.

E. DECLINED

- conservar la decisión;
- permitir que la nueva revisión vuelva a recomendar el ticker si corresponde.

Una aportación nunca puede reescribir una operación que ya ocurrió en Wio.

==================================================
13. VERSIONADO DE REVISIONES
==================================================

No sobrescribir la revisión anterior.

Persistir en cada review run:

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

Razón:

CAPITAL_CONTRIBUTION_CONFIRMED

La revisión anterior queda disponible para auditoría, pero deja de ser
operativa para recomendaciones no ejecutadas.

==================================================
14. RECÁLCULO DESDE CERO
==================================================

Crear:

recalculateGrowthRecommendationsAfterCapitalContribution

Entrada:

- account/portfolio;
- contribution IDs;
- confirmed contribution total;
- post-contribution reconciled cash;
- holdings reales;
- NAV;
- latest valid price basis;
- target allocation policy;
- target market session.

El servicio debe:

1. comprobar idempotencia;
2. confirmar que las aportaciones están disponibles;
3. comprobar que los fills conocidos están registrados;
4. resolver la sesión aplicable;
5. capturar holdings;
6. capturar cash;
7. capturar NAV;
8. localizar revisión vigente;
9. clasificar recomendaciones;
10. preservar fills ejecutados;
11. superseder pendientes no ejecutadas;
12. recalcular la cartera completa;
13. crear review run nuevo;
14. generar recomendaciones por ticker;
15. vincular revisiones;
16. publicar cuando sea operativamente seguro.

No calcular simplemente una diferencia proporcional sobre las recomendaciones
anteriores.

==================================================
15. CAPITAL Y CANTIDADES OBJETIVO
==================================================

Ejemplo:

capital_before =
50.000

confirmed_contribution =
10.000

capital_after =
60.000

La nueva revisión utiliza:

capital_basis =
60.000

cash_basis =
cash reconciliado después de la aportación

Para cada ticker:

recommended_trade_quantity =
target_quantity_at_updated_capital
-
current_actual_quantity

No utilizar:

old_recommended_quantity
+ proportional_capital_adjustment

Recalcular desde:

- capital nuevo;
- holdings reales;
- cash real;
- política de selección;
- política de sizing;
- precios de referencia sellados.

==================================================
16. AUSENCIA DE CASH ADICIONAL
==================================================

Si Omar modifica el capital objetivo, pero no confirma una aportación
disponible:

- no aumentar cash;
- no generar compras financiadas;
- no declarar mayor capacidad;
- no recalcular usando dinero inexistente.

La UI debe explicar:

“El capital objetivo cambió, pero no existe una aportación confirmada y
disponible. La revisión conserva el cash económico actual.”

Si existe aportación confirmada:

“Aportación registrada. La próxima revisión utilizará el nuevo cash
disponible.”

==================================================
17. UI
==================================================

Eliminar:

PROPUESTA DE RESIZING POR CAPITAL

Añadir una acción administrativa clara:

REGISTRAR APORTACIÓN DE CAPITAL

Después de confirmar mostrar:

- importe aportado;
- fecha de disponibilidad;
- cash anterior;
- cash posterior;
- NAV anterior;
- NAV posterior;
- sesión objetivo;
- estado de recalibración.

Estados visibles:

- APORTACIÓN PENDIENTE;
- APORTACIÓN DISPONIBLE;
- RECÁLCULO PROGRAMADO;
- RECÁLCULO EN CURSO;
- NUEVA REVISIÓN DISPONIBLE;
- RECÁLCULO DIFERIDO POR CUTOFF;
- CONCILIACIÓN DE CASH REQUERIDA.

En la revisión anterior:

“Revisión sustituida por aportación de capital.”

En la nueva:

- capital anterior;
- aportación;
- capital actualizado;
- cash disponible;
- revisión anterior;
- sesión aplicable;
- fecha de generación.

==================================================
18. IDEMPOTENCIA Y CONCURRENCIA
==================================================

Probar:

- doble confirmación de aportación;
- mismo aporte con misma referencia Wio;
- dos aportaciones antes del recálculo;
- aportación mientras se genera una revisión;
- aportación con mercado abierto;
- aportación durante premarket;
- aportación cerca del cutoff;
- formulario de recomendación anterior abierto;
- fill Wio pendiente de registro;
- reinicio antes del recálculo;
- reinicio después del recálculo;
- dos workers generando la misma revisión.

Una aportación solo puede contabilizarse una vez.

Dos aportaciones independientes:

- se registran separadamente;
- se consolidan para la siguiente revisión si comparten ventana;
- mantienen lineage individual.

Un input económico sellado debe producir una única revisión canónica.

==================================================
19. RELACIÓN CON EL HOTFIX WIO
==================================================

No confundir:

A. FILL WIO

- compra/venta ya ejecutada;
- modifica holding;
- modifica cash;
- tiene comisión;
- debe registrarse exactamente.

B. APORTACIÓN

- entrada externa de cash;
- no modifica holdings;
- no tiene ticker;
- no es compra;
- no es venta;
- no es recomendación.

C. CAMBIO DE CAPITAL OBJETIVO

- configuración;
- no crea cash;
- no modifica Libro Mayor por sí solo.

Antes de ejecutar una recalibración productiva:

- cerrar la reintroducción de fills;
- eliminar accepted orphaned falsos;
- reconciliar holdings;
- reconciliar Book;
- determinar el cash base.

==================================================
20. TRATAMIENTO DE LAS TRES CRP HUÉRFANAS
==================================================

Auditar las tres decisiones crp:

- AAPL;
- ABNB;
- AAPL.

Determinar si representan:

A. una operación Wio real;

o:

B. una propuesta abstracta de resizing sin operación.

Si B:

- no pedir datos de fill;
- no mostrar VOLVER A INTRODUCIR EJECUCIÓN WIO;
- marcar LEGACY_NON_ECONOMIC_CAPITAL_RESIZING_DECISION;
- quitar de la cola de 21 fills pendientes;
- no crear movimiento;
- no modificar cash;
- no modificar holding.

Si alguna representa una operación real:

- mantenerla en la cola post-trade;
- exigir datos reales;
- no inferirlos.

Entregar:

CRP_AUDITED =
3/3

CRP_REAL_WIO_EXECUTIONS =
<número>

CRP_NON_ECONOMIC_DECISIONS =
<número>

CRP_REMOVED_FROM_WIO_REENTRY_QUEUE =
<número>

==================================================
21. PRUEBAS
==================================================

Crear:

verify-growth-capital-contribution-accounting.mjs
verify-growth-capital-contribution-neutrality.mjs
verify-growth-capital-contribution-recalculation.mjs
verify-growth-capital-contribution-market-closed.mjs
verify-growth-capital-contribution-market-open.mjs
verify-growth-capital-contribution-cutoff.mjs
verify-growth-review-supersession.mjs
verify-growth-executed-fill-preservation.mjs
verify-growth-capital-contribution-idempotency.mjs
verify-growth-capital-contribution-ui.mjs
verify-growth-remove-capital-resizing-proposal.mjs

Casos obligatorios:

A. Aportación confirmada aumenta cash.
B. Aportación confirmada aumenta NAV.
C. Aportación no genera rentabilidad.
D. Cambio de capital objetivo sin aporte no aumenta cash.
E. Mercado cerrado recalcula para próxima apertura.
F. Mercado abierto recalcula antes de la sesión siguiente.
G. Fin de semana resuelve siguiente sesión.
H. Festivo resuelve siguiente sesión.
I. Cutoff difiere una revisión insegura.
J. Revisión anterior queda versionada.
K. Recomendaciones pendientes quedan superseded.
L. Fill Wio ejecutado permanece intacto.
M. Movimiento registrado permanece intacto.
N. Formulario antiguo queda obsoleto.
O. Dos aportaciones no se duplican.
P. Nueva revisión utiliza capital actualizado.
Q. Nueva revisión utiliza cash real.
R. Cantidades se recalculan desde cero.
S. Panel de resizing desaparece.
T. Decisiones crp no económicas no aparecen como fills.
U. No se envían órdenes a Wio.
V. Gate 2F permanece inactivo.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- hotfix Wio;
- growth core;
- recommendation engine;
- Book;
- Libro Mayor;
- cash;
- NAV;
- holdings;
- calendar;
- repository integrity.

Los seis fallos preexistentes del test capital-resizing-phase2 deben
diagnosticarse y corregirse correctamente.

No aceptarlos permanentemente como ruido si este cambio afecta precisamente a
capital resizing.

Actualizar los fixtures para aislar:

- cartera limpia;
- movimientos productivos;
- aportaciones;
- fills Wio.

==================================================
22. PRODUCCIÓN
==================================================

Antes de desplegar:

- completar reintroducción de fills o bloquear el recálculo productivo;
- backup;
- SHA-256;
- integrity_check;
- foreign_key_check;
- baseline;
- pruebas sobre copia exacta;
- Chrome;
- preview de eliminación del panel;
- auditoría crp.

Desplegar:

1. contabilidad de aportación;
2. scheduler de recálculo;
3. versionado de revisión;
4. supersession;
5. UI de aportación;
6. eliminación del panel de resizing;
7. reclasificación crp.

No crear una aportación real de prueba en Producción.

No generar una nueva revisión productiva hasta que exista una aportación real
confirmada.

==================================================
23. CHROME
==================================================

Validar:

A. El panel “Propuesta de resizing por capital” no existe.
B. Existe “Registrar aportación de capital”.
C. Cambiar capital objetivo no crea cash.
D. Confirmar aportación muestra efecto sobre cash y NAV.
E. Mercado cerrado muestra próxima apertura.
F. Mercado abierto muestra siguiente sesión.
G. Revisión anterior aparece sustituida.
H. Nueva revisión muestra nuevo capital y cash.
I. Fill Wio antiguo permanece registrable.
J. Decisión crp no económica no aparece como fill.
K. No existen errores JavaScript.
L. No se envían órdenes a Wio.

==================================================
24. COMMIT
==================================================

Crear un único commit local:

fix(growth): recalculate reviews after confirmed capital contributions

No crear tag.
No hacer push.

==================================================
25. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE =
b9bfa01

CANONICAL_HEAD_AFTER =
<hash>

SCHEMA_BEFORE =
v30

SCHEMA_AFTER =
<v30/NEXT_AVAILABLE_SCHEMA_VERSION>

GENERIC_CAPITAL_RESIZING_PROPOSAL_REMOVED =
PASS/FAIL

CAPITAL_TARGET_CHANGE_CREATES_CASH =
NO

CONFIRMED_CONTRIBUTION_CREATES_CASH =
PASS/FAIL

CAPITAL_CONTRIBUTION_LEDGER_MOVEMENT =
PASS/FAIL

CAPITAL_CONTRIBUTION_INCREASES_NAV =
PASS/FAIL

CAPITAL_CONTRIBUTION_RETURN_NEUTRAL =
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
<número>

EXECUTED_WIO_FILLS_PRESERVED =
PASS/FAIL

RECORDED_RECOMMENDATIONS_PRESERVED =
PASS/FAIL

NEW_CAPITAL_BASIS =
<importe>

NEW_CASH_BASIS =
<importe>

NEW_REVIEW_RUN_ID =
<id/NONE_WITHOUT_REAL_CONTRIBUTION>

CRP_AUDITED =
3/3

CRP_REAL_WIO_EXECUTIONS =
<número>

CRP_NON_ECONOMIC_DECISIONS =
<número>

CRP_REMOVED_FROM_WIO_REENTRY_QUEUE =
<número>

WIO_REENTRY_RECOMMENDATIONS_REMAINING =
<número>

CAPITAL_RESIZING_TESTS =
<PASS/FAIL>

CHROME_GREEN =
PASS/FAIL

JAVASCRIPT_ERRORS =
<número>

NO_BROKER_ORDERS_SENT =
PASS/FAIL

GATE_2F_REMAINS_INACTIVE =
PASS/FAIL

SECRET_SCAN =
PASS/FAIL

NO_PUSH_PERFORMED =
PASS/FAIL

LOCAL_COMMIT =
<hash>

Detenerse después de eliminar el resizing abstracto y habilitar la
recalibración automática por aportaciones reales.