MIZAN · CARTERA DE CRECIMIENTO
FASE 2 DEFINITIVA
APORTACIÓN CONFIRMADA → RECÁLCULO AUTOMÁTICO DE RECOMENDACIONES
CALENDARIO NYSE · VERSIONADO · SUPERSESSION · SCHEDULER · CATCH-UP

Esta orden parte del cierre aprobado de Fase 1.

No reimplementar el resizing eliminado.
No reabrir el incidente Wio.
No modificar Gate 2F.
No crear aportaciones productivas ficticias.
No enviar órdenes a Wio.

==================================================
0. BASELINE OBLIGATORIO
==================================================

CANONICAL_HEAD esperado =
ed9282c

SCHEMA esperado =
v30

Estado confirmado:

GROWTH_BOOK_RECONCILIATION_STATUS =
RECONCILED

WIO_REENTRY_RECOMMENDATIONS_REMAINING =
0

SESSION_6_RECOMMENDATIONS_WITH_MOVEMENT =
26/26

DECISIONS_WITHOUT_MOVEMENT =
0

DUPLICATE_MOVEMENTS =
0

PARTIAL_WRITES =
0

PANW_MOVEMENT_ID =
91

GENERIC_CAPITAL_RESIZING_PROPOSAL_REMOVED =
PASS

NEW_CRP_RECOMMENDATIONS_POSSIBLE =
NO

LEGACY_CRP_DECISIONS_PRESERVED =
3/3

CRP_REAL_WIO_EXECUTIONS =
0

CONFIRMED_CONTRIBUTION_INCREASES_CASH =
PASS

CONFIRMED_CONTRIBUTION_INCREASES_NAV =
PASS

CAPITAL_CONTRIBUTION_RETURN_NEUTRAL =
PASS

CAPITAL_TARGET_CHANGE_CREATES_CASH =
NO

Gate 2F:

INACTIVE

Antes de modificar:

- confirmar HEAD;
- confirmar schema;
- comprobar git status;
- inventariar untracked;
- confirmar una sola instancia;
- comprobar /ping;
- comprobar env-info;
- integrity_check;
- foreign_key_check;
- backup SQLite;
- SHA-256;
- validar backup.

Si HEAD no es ed9282c:

- inventariar el estado;
- no resetear;
- no descartar cambios;
- continuar solo si el estado es coherente.

==================================================
1. VALIDACIÓN CHROME DE FASE 1
==================================================

Antes de construir Fase 2, validar en Producción:

- el panel “Propuesta de resizing por capital” no existe;
- no existe el botón “Recalcular cartera” asociado al resizing;
- las tres decisiones crp no aparecen operativamente;
- no aparecen en la cola Wio;
- no aparecen como movimientos;
- “Aportar capital” sigue disponible;
- no hay errores JavaScript;
- Book, holdings, cash y NAV cargan correctamente.

Declarar:

PHASE_1_CHROME_VALIDATION =
PASS/FAIL

JAVASCRIPT_ERRORS =
<número>

No continuar si la retirada del resizing dejó rota la UI.

==================================================
2. OBJETIVO FUNCIONAL
==================================================

Completar el flujo:

APORTACIÓN DISPONIBLE EN WIO
→ REGISTRO EN MIZAN
→ MOVIMIENTO EXTERNAL_CAPITAL_CONTRIBUTION
→ AUMENTO DE CASH
→ AUMENTO DE NAV
→ RENTABILIDAD NEUTRAL
→ SOLICITUD DE RECÁLCULO
→ RESOLUCIÓN DE SESIÓN NYSE
→ VERSIONADO DE LA REVISIÓN
→ SUPERSESSION DE PENDIENTES
→ RECÁLCULO DESDE ESTADO CANÓNICO COMPLETO
→ NUEVAS RECOMENDACIONES NORMALES

Mizan no ejecuta operaciones en Wio.

Las recomendaciones resultantes se ejecutan manualmente en Wio y después se
registran como fills reales mediante el flujo post-trade existente.

==================================================
3. REUTILIZAR LA CONTABILIDAD EXISTENTE
==================================================

Code confirmó que la contabilidad de aportaciones ya existe y es correcta.

Antes de añadir código, identificar y documentar:

- endpoint actual de “Aportar capital”;
- servicio económico utilizado;
- tabla o tablas afectadas;
- tipo de movimiento;
- cash event;
- neutralización de rentabilidad;
- cálculo de NAV;
- idempotencia existente;
- validaciones existentes;
- UI existente.

No crear una segunda implementación paralela.

Extender el flujo actual únicamente para añadir:

- contribution identity durable;
- available_at;
- recalculation request;
- sesión objetivo;
- estado de recálculo;
- lineage con la revisión nueva;
- idempotencia extremo a extremo.

==================================================
4. APORTACIÓN CONFIRMADA
==================================================

Una aportación solo puede utilizarse cuando Omar confirma:

“Este importe ya está disponible para operar en Wio.”

Datos obligatorios:

- portfolio/account;
- amount;
- currency;
- available_date;
- available_time nullable;
- Wio reference nullable;
- notes nullable;
- idempotency key;
- confirmed_by;
- confirmed_at.

Estados mínimos:

- DRAFT;
- PENDING_AVAILABILITY;
- CONFIRMED_AVAILABLE;
- CANCELLED;
- REJECTED.

Solo CONFIRMED_AVAILABLE:

- crea movimiento;
- aumenta cash;
- aumenta NAV;
- crea una solicitud de recálculo.

Una transferencia pendiente no aumenta capacidad.

==================================================
5. PREVIEW
==================================================

Antes de confirmar mostrar:

- importe;
- moneda;
- cash actual;
- cash proyectado;
- NAV actual;
- NAV proyectado;
- holdings sin cambios;
- mercado USA abierto/cerrado;
- próxima sesión;
- sesión objetivo;
- cutoff;
- revisión vigente;
- recomendaciones pendientes afectadas;
- reconciliation status;
- preview hash.

El preview:

- no escribe;
- no aumenta cash;
- no modifica NAV;
- no crea revisión;
- no supersede recomendaciones.

==================================================
6. REGISTRO ATÓMICO
==================================================

La confirmación debe ejecutarse en una sola transacción:

1. validar Book reconciliado;
2. validar preview;
3. validar importe y moneda;
4. validar available_at;
5. validar idempotencia;
6. detectar duplicados;
7. registrar contribution detail;
8. crear movimiento EXTERNAL_CAPITAL_CONTRIBUTION;
9. crear cash event;
10. aumentar cash;
11. aumentar NAV;
12. preservar neutralidad de rentabilidad;
13. crear recalculation request;
14. registrar audit event;
15. COMMIT.

Debe cumplirse:

cash_after =
cash_before + contribution_amount

NAV_after =
NAV_before + contribution_amount

holdings_after =
holdings_before

commission =
0

return_created =
0

Si falla cualquier paso:

- ROLLBACK completo;
- no cambia cash;
- no cambia NAV;
- no se crea solicitud de recálculo.

==================================================
7. CAPITAL OBJETIVO
==================================================

Mantener esta invariante:

CAPITAL_TARGET_CHANGE_CREATES_CASH =
NO

Cambiar capital objetivo:

- no crea movimiento;
- no aumenta cash;
- no aumenta NAV;
- no financia compras;
- no dispara por sí solo una revisión económica.

Solo una aportación CONFIRMED_AVAILABLE crea dinero disponible.

No reintroducir directa ni indirectamente:

- capital_resizing_proposal;
- capital_resizing_line;
- crp;
- recomendaciones abstractas de resizing.

==================================================
8. CALENDARIO NYSE
==================================================

Usar el calendario canónico y:

America/New_York

No usar una comprobación simplificada de día laborable.

Resolver:

- mercado regular abierto;
- mercado regular cerrado;
- premarket;
- after-hours;
- fin de semana;
- festivo;
- early close;
- cutoff preopen.

A. APORTACIÓN CON MERCADO CERRADO

Registrar inmediatamente la aportación.

Programar el recálculo para antes de la próxima apertura si existe margen
operativo suficiente.

Estado:

RECALCULATION_SCHEDULED_FOR_NEXT_OPEN

B. APORTACIÓN CON MERCADO ABIERTO

Registrar inmediatamente la aportación.

No publicar una revisión nueva durante la sesión.

Programar el recálculo para antes de la apertura de la siguiente sesión.

Estado:

RECALCULATION_SCHEDULED_FOR_NEXT_SESSION_PREOPEN

C. CUTOFF

Si no hay tiempo seguro para:

- sellar holdings;
- resolver cash;
- obtener precios;
- calcular;
- validar;
- publicar;

usar:

RECALCULATION_DEFERRED_BY_CUTOFF

y resolver la siguiente ventana segura.

==================================================
9. ESTADOS DEL RECÁLCULO
==================================================

Estados mínimos:

- REQUESTED;
- SCHEDULED;
- WAITING_FOR_MARKET_WINDOW;
- WAITING_FOR_PRICES;
- READY;
- RUNNING;
- COMPLETED;
- DEFERRED_BY_CUTOFF;
- FAILED_RETRYABLE;
- FAILED_FINAL;
- CANCELLED;
- SUPERSEDED.

Toda transición debe quedar auditada.

No considerar completado un recálculo hasta que exista una revisión nueva
válida y vinculada.

==================================================
10. SNAPSHOT DE INPUTS
==================================================

Antes del recálculo sellar:

- holdings reales;
- cash reconciliado;
- NAV;
- aportaciones incluidas;
- política de selección;
- política de sizing;
- precios;
- sesión de precios;
- sesión aplicable;
- revisión vigente;
- input hash.

No utilizar:

- recomendaciones anteriores como cantidades base;
- capital objetivo como cash;
- fills no registrados;
- ventas recomendadas;
- proceeds estimados;
- precios live no certificados.

==================================================
11. VERSIONADO DE REVISIONES
==================================================

Persistir:

- review_run_id;
- capital_basis;
- cash_basis;
- NAV_basis;
- holdings snapshot;
- contribution IDs;
- applicable market session;
- generated_at;
- price basis;
- price session;
- input hash;
- supersedes_review_run_id;
- superseded_by_review_run_id;
- supersession_reason.

Razón:

CAPITAL_CONTRIBUTION_CONFIRMED

No sobrescribir ni borrar la revisión anterior.

==================================================
12. SUPERSESSION
==================================================

Tratar las recomendaciones de la revisión anterior así:

PENDING_NOT_EXECUTED:

- SUPERSEDED_BY_CAPITAL_CHANGE;
- deja de ser operativa;
- se crea reemplazo en la revisión nueva.

FORM_OPEN_NO_WIO_EXECUTION:

- marcar obsoleta;
- conservar datos del formulario;
- impedir confirmación contra la revisión antigua;
- ofrecer la recomendación nueva.

EXECUTED_IN_WIO_PENDING_REGISTRATION:

- no modificar;
- no invalidar;
- permitir registrar el fill exacto;
- conservar lineage original.

ACCEPTED_AND_RECORDED:

- no modificar;
- no revertir;
- conservar movimiento.

DECLINED:

- conservar historia;
- la nueva revisión puede recomendar nuevamente el ticker.

La supersession debe ser atómica con la publicación de la nueva revisión.

Si el recálculo falla, la revisión anterior no debe quedar inutilizada a
medias.

==================================================
13. RECÁLCULO DESDE ESTADO CANÓNICO
==================================================

Crear:

recalculateGrowthRecommendationsAfterCapitalContribution

Debe:

1. validar reconciliation gate;
2. comprobar idempotencia;
3. cargar contribution request;
4. resolver sesión;
5. cargar snapshot sellado;
6. cargar cash y NAV;
7. cargar holdings reales;
8. cargar precios certificados;
9. localizar revisión vigente;
10. preservar operaciones ejecutadas;
11. recalcular la cartera completa;
12. crear review run nuevo;
13. crear recomendaciones normales;
14. superseder pendientes;
15. vincular revisiones;
16. registrar estado COMPLETED;
17. COMMIT.

No hacer un ajuste proporcional sobre recomendaciones anteriores.

==================================================
14. RECOMENDACIONES RESULTANTES
==================================================

Generar exclusivamente:

- MANTENER;
- INCORPORAR;
- AUMENTAR;
- REDUCIR;
- ELIMINAR.

No generar:

- RESIZING;
- CAPITAL_RESIZING;
- CRP;
- una línea genérica por el importe aportado.

Por ticker:

recommended_trade_quantity =
target_quantity_at_updated_capital
-
current_actual_quantity

La base es:

- holdings reales;
- cash real;
- NAV actualizado;
- capital económico actualizado;
- política vigente;
- precios sellados.

==================================================
15. MÚLTIPLES APORTACIONES
==================================================

Si existen dos aportaciones confirmadas antes de ejecutar el recálculo:

- registrar cada flujo por separado;
- conservar lineage individual;
- consolidar ambas en el snapshot económico;
- producir una única revisión canónica si comparten la misma ventana;
- marcar las recalculation requests absorbidas o superseded;
- no duplicar recomendaciones.

==================================================
16. SCHEDULER
==================================================

Integrar con el scheduler canónico existente.

No crear otro scheduler.

Debe:

- localizar recálculos pendientes;
- resolver la ventana;
- esperar al cutoff correcto;
- esperar precios certificados;
- procesar cronológicamente;
- ejecutar una sola vez;
- aislar errores por cartera;
- no enviar órdenes.

Un fallo de recálculo no puede bloquear:

- Book;
- registro de fills;
- Track;
- otras tareas no relacionadas.

==================================================
17. STARTUP CATCH-UP
==================================================

Implementar recuperación de:

- servidor apagado antes de la ventana;
- reinicio antes de COMMIT;
- reinicio después de COMMIT;
- WAITING_FOR_PRICES;
- FAILED_RETRYABLE;
- revisión creada pero estado no actualizado;
- aportaciones múltiples pendientes.

Catch-up debe:

- detectar el resultado ya existente;
- no duplicar revisiones;
- no duplicar recomendaciones;
- no volver a contabilizar la aportación;
- no enviar órdenes.

==================================================
18. IDEMPOTENCIA Y CONCURRENCIA
==================================================

Probar:

- doble clic de confirmación;
- misma referencia Wio;
- mismo idempotency key;
- payload distinto con misma key;
- dos workers;
- scheduler y catch-up;
- dos aportaciones;
- recálculo concurrente;
- formulario antiguo abierto;
- reinicio en puntos críticos.

Resultados:

Mismo aporte:

ALREADY_RECORDED

Mismo key con payload distinto:

IDEMPOTENCY_CONFLICT

Mismo input económico:

- una revisión;
- un conjunto de recomendaciones;
- cero duplicados.

Usar transacciones y bloqueo equivalentes al patrón canónico del repositorio.

==================================================
19. UI
==================================================

Conservar:

APORTAR CAPITAL

o renombrar claramente:

REGISTRAR APORTACIÓN DE CAPITAL

Mostrar:

- importe;
- moneda;
- disponibilidad;
- referencia Wio;
- cash anterior;
- cash posterior;
- NAV anterior;
- NAV posterior;
- mercado abierto/cerrado;
- sesión objetivo;
- cutoff;
- estado de recálculo.

Estados visibles:

- APORTACIÓN PENDIENTE;
- APORTACIÓN REGISTRADA;
- RECÁLCULO PROGRAMADO;
- ESPERANDO PRECIOS;
- RECÁLCULO EN CURSO;
- NUEVA REVISIÓN DISPONIBLE;
- DIFERIDO POR CUTOFF;
- ERROR RECUPERABLE.

En la revisión anterior:

“Revisión sustituida por aportación de capital.”

En la nueva revisión mostrar:

- capital anterior;
- total aportado;
- capital actualizado;
- cash disponible;
- NAV;
- revisión sustituida;
- sesión;
- precios usados;
- fecha de generación.

==================================================
20. ENDPOINTS
==================================================

Crear o completar:

- preview contribution;
- create contribution draft;
- confirm contribution available;
- cancel draft/pending contribution;
- get contribution;
- list contributions;
- get recalculation status;
- retry recalculation when safe.

Toda escritura debe:

- validar idempotencia;
- validar Book reconciliado;
- usar transacción;
- devolver contribution_id;
- devolver movement_id;
- devolver recalculation_request_id;
- devolver sesión y estado.

No crear endpoints para operar en Wio.

==================================================
21. SCHEMA
==================================================

Auditar primero el schema existente.

Si v30 soporta todo el contrato:

- no migrar.

Si falta soporte durable:

- utilizar NEXT_AVAILABLE_SCHEMA_VERSION;
- migración aditiva mínima;
- no reutilizar tablas crp;
- no modificar historia;
- no insertar aportaciones;
- no generar revisiones durante la migración;
- probar copia exacta antes de Producción.

Tablas/campos deben soportar:

- contribution status;
- available_at;
- broker reference;
- idempotency;
- recalculation request;
- state transitions;
- target session;
- cutoff;
- contribution-review lineage;
- review supersession;
- input hash;
- errores y retries.

==================================================
22. PRUEBAS
==================================================

Crear suites específicas para:

- preview sin escritura;
- aportación contable;
- cash;
- NAV;
- neutralidad de rentabilidad;
- holdings sin cambios;
- capital objetivo sin cash;
- mercado cerrado;
- mercado abierto;
- premarket;
- fin de semana;
- festivo;
- early close;
- cutoff;
- snapshot de inputs;
- recalculation from canonical state;
- supersession;
- preservación de fills;
- preservación de movimientos;
- múltiples aportaciones;
- idempotencia;
- concurrencia;
- scheduler;
- catch-up;
- UI;
- eliminación permanente de crp.

Casos obligatorios:

A. Fase 1 sigue verde.
B. No reaparece el panel.
C. No pueden generarse crp.
D. Preview no escribe.
E. Confirmación crea movimiento.
F. Cash aumenta exactamente.
G. NAV aumenta exactamente.
H. Holdings no cambian.
I. Rentabilidad no cambia.
J. Capital objetivo no crea cash.
K. Mercado cerrado usa próxima apertura.
L. Mercado abierto usa siguiente sesión.
M. Cutoff difiere.
N. Festivo y fin de semana resuelven correctamente.
O. Revisión anterior queda versionada.
P. Pendientes se superseden atómicamente.
Q. Fills ejecutados se preservan.
R. Movimientos registrados se preservan.
S. Recálculo usa estado canónico completo.
T. Solo se crean recomendaciones normales.
U. Dos aportaciones pueden consolidarse.
V. Doble confirmación no duplica cash.
W. Doble recálculo no duplica revisión.
X. Catch-up recupera el trabajo.
Y. No se envían órdenes a Wio.
Z. Gate 2F continúa inactivo.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

Ejecutar además:

- verify-growth-remove-capital-resizing;
- growth-core-end-to-end;
- financial-core;
- Book;
- Libro Mayor;
- holdings;
- cash;
- NAV;
- recommendation engine;
- calendar;
- scheduler;
- startup catch-up;
- migration;
- repository integrity.

No reintroducir la suite eliminada tal como estaba.

Crear cobertura nueva alineada con el nuevo contrato de aportación y recálculo.

==================================================
23. VALIDACIÓN LAB
==================================================

En LAB demostrar al menos:

A. APORTACIÓN CON MERCADO CERRADO

- registrar fixture de aportación;
- cash y NAV aumentan;
- rentabilidad neutral;
- recálculo para próxima apertura;
- revisión nueva;
- pendientes superseded.

B. APORTACIÓN CON MERCADO ABIERTO

- cash y NAV aumentan;
- no se publica revisión intradía;
- recálculo para siguiente preopen.

C. CUTOFF

- no publica revisión insegura;
- difiere correctamente.

D. REINICIO

- catch-up recupera;
- cero duplicados.

No usar datos productivos reales como fixtures mutables.

==================================================
24. PREVIEW PRODUCTIVO
==================================================

Antes de desplegar:

- backup;
- SHA-256;
- integrity_check;
- foreign_key_check;
- migrar copia si corresponde;
- ejecutar todas las suites;
- iniciar servidor en copia;
- validar scheduler;
- validar catch-up;
- validar Chrome;
- comparar Book;
- comparar holdings;
- comparar cash;
- comparar NAV.

No desplegar si:

- una aportación genera rentabilidad;
- modifica holdings;
- duplica cash;
- crea revisión en una sesión incorrecta;
- reaparecen crp;
- la supersession puede quedar parcial;
- se envían órdenes.

==================================================
25. DESPLIEGUE
==================================================

Desplegar código, schema y UI únicamente después de preview verde.

El despliegue no debe:

- crear aportaciones;
- crear movimientos;
- aumentar cash;
- aumentar NAV;
- generar revisiones;
- superseder recomendaciones;
- enviar órdenes.

Verificar después:

PRODUCTION_CONTRIBUTIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_REVIEWS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_RECOMMENDATIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_BROKER_ORDERS_SENT =
0

La funcionalidad queda disponible para la próxima aportación real registrada
por Omar.

==================================================
26. CHROME
==================================================

Validar sin crear una aportación productiva ficticia:

- resizing sigue ausente;
- crp siguen fuera de operación;
- “Registrar aportación” existe;
- preview funciona;
- cash y NAV proyectados se muestran;
- estado de mercado se muestra;
- sesión objetivo se muestra;
- capital objetivo no muestra cash nuevo;
- no hay errores JavaScript.

Utilizar LAB o fixture aislado para validar la confirmación económica y la
revisión nueva.

==================================================
27. GATE 2F
==================================================

Gate 2F permanece aparcado.

No modificar:

- policy status;
- effective_from;
- reservas prospectivas;
- endpoints de La Lente;
- UI prospectiva.

Resultado:

GATE_2F_REMAINS_INACTIVE =
PASS

==================================================
28. COMMIT
==================================================

Crear un único commit local:

feat(growth): recalculate reviews after confirmed capital contributions

No hacer push.
No crear tag.

Ejecutar secret scan.

==================================================
29. ENTREGA FINAL
==================================================

CANONICAL_HEAD_BEFORE =
ed9282c

CANONICAL_HEAD_AFTER =
<hash>

SCHEMA_BEFORE =
v30

SCHEMA_AFTER =
<v30/NEXT_AVAILABLE_SCHEMA_VERSION>

PHASE_1_CHROME_VALIDATION =
PASS/FAIL

GENERIC_CAPITAL_RESIZING_PROPOSAL_REMOVED =
PASS

NEW_CRP_RECOMMENDATIONS_POSSIBLE =
NO

LEGACY_CRP_DECISIONS_PRESERVED =
3/3

CAPITAL_CONTRIBUTION_EXISTING_ACCOUNTING_REUSED =
PASS/FAIL

CAPITAL_CONTRIBUTION_PREVIEW =
PASS/FAIL

CAPITAL_CONTRIBUTION_MOVEMENT =
PASS/FAIL

CONFIRMED_CONTRIBUTION_INCREASES_CASH =
PASS/FAIL

CONFIRMED_CONTRIBUTION_INCREASES_NAV =
PASS/FAIL

CAPITAL_CONTRIBUTION_RETURN_NEUTRAL =
PASS/FAIL

CAPITAL_CONTRIBUTION_HOLDINGS_UNCHANGED =
PASS/FAIL

CAPITAL_TARGET_CHANGE_CREATES_CASH =
NO

MARKET_CLOSED_TARGET_SESSION =
PASS/FAIL

MARKET_OPEN_TARGET_SESSION =
PASS/FAIL

PREOPEN_CUTOFF =
PASS/FAIL

HOLIDAY_AND_WEEKEND_RESOLUTION =
PASS/FAIL

EARLY_CLOSE_RESOLUTION =
PASS/FAIL

CANONICAL_INPUT_SNAPSHOT =
PASS/FAIL

RECALCULATION_FROM_CANONICAL_STATE =
PASS/FAIL

OLD_REVIEW_VERSIONED =
PASS/FAIL

PENDING_RECOMMENDATIONS_SUPERSEDED =
PASS/FAIL

EXECUTED_WIO_FILLS_PRESERVED =
PASS/FAIL

RECORDED_MOVEMENTS_PRESERVED =
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

PRODUCTION_RECOMMENDATIONS_CREATED_BY_DEPLOYMENT =
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

Detenerse después de completar, desplegar y verificar Fase 2.
No crear una aportación productiva de prueba.