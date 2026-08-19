MIZAN · CARTERA DE CRECIMIENTO
FASE 2B DEFINITIVA
SNAPSHOT CANÓNICO · RECÁLCULO · VERSIONADO · SUPERSESSION
SCHEDULER · CATCH-UP · UI · MIGRACIÓN v31 · DESPLIEGUE

SESIÓN NUEVA · CONTEXTO FRESCO

Esta sesión está dedicada exclusivamente a completar Fase 2B.

Fase 2A está completa y commiteada.

No reimplementar Fase 2A.
No crear otra ruta económica.
No reintroducir resizing ni crp.
No modificar Gate 2F.
No enviar órdenes a Wio.
No crear aportaciones productivas ficticias.
No generar revisiones productivas ficticias.

==================================================
0. BASELINE OBLIGATORIO
==================================================

BRANCH esperada =
release/defensiva-operational-parity-v1.1

CANONICAL_HEAD esperado =
1bf6a8b

BASELINE_ANTERIOR =
ed9282c

SCHEMA DE PRODUCCIÓN =
v30

SCHEMA DEL CÓDIGO =
v31

PRODUCTION_V31_DEPLOYED =
FALSE

Fase 2A entregó:

- capital_contribution;
- financial_recalculation_request;
- financial_recalculation_revision;
- migrateV31;
- previewCapitalContribution;
- confirmCapitalContribution;
- resolveContributionSession;
- POST /financial/:cartera/capital-contribution;
- endpoint preview;
- listado de aportaciones;
- listado de recalculation requests;
- reutilización de FC.recordCashEvent("CAPITAL_CONTRIBUTION");
- cash y NAV correctos;
- rentabilidad neutral;
- idempotencia;
- 29/0 unit;
- 16/0 HTTP.

Estado productivo:

GROWTH_BOOK_RECONCILIATION_STATUS =
RECONCILED

WIO_REENTRY_RECOMMENDATIONS_REMAINING =
0

GENERIC_CAPITAL_RESIZING_PROPOSAL_REMOVED =
PASS

NEW_CRP_RECOMMENDATIONS_POSSIBLE =
NO

LEGACY_CRP_DECISIONS_PRESERVED =
3/3

GATE_2F_REMAINS_INACTIVE =
TRUE

Antes de modificar:

- confirmar branch;
- confirmar HEAD;
- confirmar schema productivo v30;
- confirmar código objetivo v31;
- revisar git status;
- inventariar archivos modificados y untracked;
- cargar y verificar mizan-capital-contribution-phase2a.md;
- comprobar una sola raíz Git;
- comprobar una sola instancia productiva;
- /ping;
- env-info;
- integrity_check;
- foreign_key_check;
- backup;
- SHA-256;
- validar el backup.

Si HEAD no es 1bf6a8b:

- no resetear;
- inventariar;
- identificar commits posteriores;
- continuar solo si son coherentes.

==================================================
1. CORRECCIÓN OBLIGATORIA DEL CUTOFF
==================================================

Fase 2A informa:

cutoff preopen =
09:30 ET

Esto no es un cutoff preopen seguro.

09:30 America/New_York es normalmente la apertura regular, no un margen
operativo anterior a la apertura.

Corregir el contrato antes del despliegue.

El cutoff debe resolverse como:

preopen_cutoff_at =
official_market_open_at
-
configured_safety_margin

El margen debe:

- ser configurable;
- ser positivo;
- estar expresado explícitamente en minutos o duración;
- no estar hardcodeado dentro del motor;
- utilizar la apertura oficial de la sesión;
- respetar festivos;
- respetar calendario NYSE;
- funcionar con cambios DST;
- poder probarse con reloj inyectable.

No asumir que early close modifica la hora de apertura.

Si no existe configuración previa, definir un valor conservador documentado,
pero no utilizar 09:30 como cutoff.

Entregar:

PREOPEN_CUTOFF_BEFORE_MARKET_OPEN =
PASS/FAIL

PREOPEN_CUTOFF_MARGIN_MINUTES =
<número>

==================================================
2. OBJETIVO DE FASE 2B
==================================================

Completar:

aportación CONFIRMED_AVAILABLE
→ financial_recalculation_request
→ resolución de ventana NYSE
→ snapshot canónico de inputs
→ recálculo desde estado económico completo
→ creación de revisión versionada
→ creación de recomendaciones normales
→ validación integral
→ supersession atómica de pendientes
→ publicación
→ COMPLETED

Añadir:

- scheduler;
- startup catch-up;
- recuperación;
- UI de estados;
- despliegue de v31;
- validación Chrome.

No contabilizar nuevamente la aportación.

La aportación ya fue registrada económicamente por Fase 2A mediante:

FC.recordCashEvent("CAPITAL_CONTRIBUTION")

Fase 2B consume la solicitud de recálculo, no vuelve a crear cash ni NAV.

==================================================
3. AUDITORÍA DE FASE 2A
==================================================

Antes de extender:

- revisar las tres tablas v31;
- revisar constraints;
- revisar foreign keys;
- revisar índices;
- revisar estados;
- revisar idempotency keys;
- revisar confirmCapitalContribution;
- revisar creación de recalculation request;
- revisar resolveContributionSession;
- revisar endpoints;
- revisar atomicidad.

Confirmar:

ONE_CONTRIBUTION_ONE_CASH_EFFECT =
PASS/FAIL

ONE_CONTRIBUTION_ONE_RECALCULATION_REQUEST =
PASS/FAIL

CONFIRM_RETRY_DOES_NOT_DUPLICATE_CASH =
PASS/FAIL

Si falta una restricción necesaria:

- completar v31 antes del despliegue;
- mantener la migración v30→v31 única;
- no crear v32 porque v31 todavía no está desplegada;
- conservar compatibilidad con Fase 2A.

==================================================
4. ESTADOS DE RECÁLCULO
==================================================

financial_recalculation_request debe soportar como mínimo:

- PENDING;
- SCHEDULED;
- WAITING_FOR_MARKET_WINDOW;
- WAITING_FOR_PRICES;
- READY;
- RUNNING;
- COMPLETED;
- DEFERRED_BY_CUTOFF;
- FAILED_RETRYABLE;
- FAILED_FINAL;
- SUPERSEDED;
- CANCELLED.

Persistir:

- contribution_id;
- portfolio/account;
- requested_at;
- target_market_session;
- target_window;
- scheduled_for;
- status;
- attempt_count;
- last_attempt_at;
- failure_code;
- failure_reason;
- input_hash;
- revision_id;
- superseded_by_request_id;
- completed_at;
- audit timestamps.

Las transiciones deben:

- validarse;
- ser idempotentes;
- quedar auditadas;
- impedir volver de COMPLETED a un estado pendiente;
- impedir dos revisiones canónicas para el mismo input.

==================================================
5. RESOLUCIÓN TEMPORAL NYSE
==================================================

Usar:

America/New_York

y el calendario NYSE canónico.

A. MERCADO CERRADO, ANTES DEL CUTOFF

- target session = próxima sesión;
- target window = preopen;
- estado = SCHEDULED.

B. PREMARKET DESPUÉS DEL CUTOFF

- no generar una revisión apresurada;
- target session = sesión siguiente a la apertura inminente;
- estado = DEFERRED_BY_CUTOFF.

C. MERCADO ABIERTO

- no publicar revisión intradía;
- target session = siguiente sesión bursátil;
- target window = preopen.

D. AFTER-HOURS

- target session = siguiente sesión;
- sujeto al cutoff de esa apertura.

E. FIN DE SEMANA/FESTIVO

- resolver la próxima sesión real.

F. EARLY CLOSE

- respetar el cierre oficial para determinar si el mercado está abierto;
- la revisión continúa destinada al próximo preopen seguro.

No usar:

- lunes-viernes simplificado;
- horas UTC fijas;
- 09:30 como cutoff;
- reloj del sistema sin abstracción.

==================================================
6. SNAPSHOT CANÓNICO DE INPUTS
==================================================

Poblar financial_recalculation_revision con un snapshot durable e inmutable.

Debe incluir o referenciar:

- portfolio/account;
- contribution IDs incluidos;
- contribution total;
- cash reconciliado;
- NAV;
- capital económico;
- holdings snapshot;
- quantities;
- Book state/version;
- policy/config version;
- selection inputs;
- sizing inputs;
- price manifest/version;
- price basis;
- price session;
- applicable market session;
- previous canonical review;
- generated_at;
- input hash;
- status;
- revision version.

No usar:

- capital objetivo como cash;
- recomendaciones antiguas como cantidades base;
- ventas recomendadas como proceeds;
- fills no registrados;
- precios live no certificados;
- crp legacy.

El snapshot debe poder reconstruir por qué se generó cada recomendación.

Una vez CANONICAL:

- no modificar;
- no sustituir inputs;
- no reescribir el hash.

==================================================
7. CONSOLIDACIÓN DE APORTACIONES
==================================================

Si existen varias aportaciones CONFIRMED_AVAILABLE antes del recálculo:

- mantener cada aportación como movimiento independiente;
- no fusionar movimientos de cash;
- incluir todas las contribution IDs elegibles;
- calcular contribution total;
- sellar un único snapshot;
- generar una única revisión si comparten la misma ventana económica;
- marcar las solicitudes absorbidas como SUPERSEDED o agrupadas según contrato;
- conservar lineage completo.

No incluir una aportación ya utilizada en otra revisión canónica.

==================================================
8. MOTOR DE RECÁLCULO
==================================================

Implementar:

recalculateGrowthRecommendationsAfterCapitalContribution

Debe:

1. adquirir bloqueo transaccional;
2. validar el request;
3. validar Book reconciliado;
4. confirmar que la aportación ya fue contabilizada;
5. confirmar que no se contabiliza de nuevo;
6. resolver sesión y ventana;
7. comprobar cutoff;
8. comprobar precios certificados;
9. crear snapshot de inputs;
10. localizar revisión operativa vigente;
11. cargar holdings reales;
12. cargar cash y NAV actuales;
13. ejecutar el motor normal de recomendaciones;
14. recalcular desde el estado canónico completo;
15. crear revisión candidata;
16. crear recomendaciones candidatas;
17. validar invariantes;
18. superseder pendientes de la revisión anterior;
19. promover revisión candidata a CANONICAL;
20. vincular contribution/request/revision;
21. marcar request COMPLETED;
22. COMMIT.

Si falla cualquier paso:

- ROLLBACK;
- revisión anterior sigue vigente;
- no queda supersession parcial;
- no queda revisión canónica incompleta;
- no se duplica la aportación;
- request queda retryable o final según la causa.

==================================================
9. REUTILIZAR EL MOTOR NORMAL
==================================================

No crear un segundo motor de selección o sizing.

Reutilizar el motor diario normal de la cartera de crecimiento.

La nueva revisión debe generar únicamente:

- MANTENER;
- INCORPORAR;
- AUMENTAR;
- REDUCIR;
- ELIMINAR.

No generar:

- CAPITAL_RESIZING;
- RESIZING;
- CRP;
- capital_resizing_line;
- recomendación genérica por el importe aportado.

Para cada ticker:

recommended_trade_quantity =
target_quantity_at_updated_capital
-
current_actual_quantity

Recalcular desde cero.

No aplicar un porcentaje sobre las recomendaciones anteriores.

==================================================
10. VERSIONADO
==================================================

Cada revisión debe tener:

- revision_id;
- revision_version;
- status;
- previous_revision_id;
- supersedes_review_run_id;
- superseded_by_review_run_id;
- supersession_reason;
- input_hash;
- content hash;
- canonical timestamp.

Estados mínimos:

- CANDIDATE;
- VALIDATED;
- CANONICAL;
- SUPERSEDED;
- FAILED.

Debe existir una sola revisión CANONICAL aplicable por:

- portfolio;
- economic input;
- applicable session;
- methodology/version.

Utilizar:

UNIQUE;
índice parcial;
validación transaccional;

según soporte SQLite.

==================================================
11. SUPERSESSION ATÓMICA
==================================================

No superseder la revisión anterior antes de validar completamente la nueva.

Orden:

1. crear candidate;
2. generar candidate recommendations;
3. validar;
4. en la misma transacción:
   - marcar la anterior como superseded;
   - marcar pendientes anteriores como superseded;
   - promover la nueva a canonical;
   - publicar recomendaciones nuevas;
   - completar request.

Si falla:

- la revisión anterior conserva su estado operativo;
- no queda una ventana sin revisión válida.

Razón:

CAPITAL_CONTRIBUTION_CONFIRMED

==================================================
12. TRATAMIENTO DE RECOMENDACIONES ANTERIORES
==================================================

PENDING_NOT_EXECUTED:

- SUPERSEDED_BY_CAPITAL_CHANGE;
- no puede aceptarse después;
- tiene vínculo a reemplazo cuando exista.

FORM_OPEN_NO_WIO_EXECUTION:

- conservar draft;
- marcar formulario obsoleto;
- impedir confirmación contra la revisión anterior;
- ofrecer abrir la nueva.

EXECUTED_IN_WIO_PENDING_REGISTRATION:

- no modificar;
- no superseder como fill;
- permitir registro post-trade exacto;
- conservar lineage a la revisión original.

ACCEPTED_AND_RECORDED:

- no modificar;
- no revertir;
- conservar movement_id.

DECLINED:

- conservar historia;
- permitir recomendación nueva si corresponde.

La supersession afecta la vigencia operativa, no borra historia.

==================================================
13. PRECIOS
==================================================

Utilizar exclusivamente la base canónica permitida por las revisiones.

Persistir:

- price basis;
- price session;
- manifest/version;
- provider/source;
- coverage;
- content hash.

Si faltan precios obligatorios:

WAITING_FOR_PRICES

No:

- publicar revisión parcial;
- usar live como fallback;
- utilizar una sesión posterior retroactivamente;
- superseder la revisión vigente.

Cuando aparezcan:

- reintentar;
- utilizar la sesión exacta;
- crear una única revisión.

==================================================
14. SCHEDULER
==================================================

Integrar con el scheduler existente.

No crear un scheduler paralelo.

Añadir un adaptador:

processPendingGrowthCapitalRecalculations

Debe:

- localizar requests pendientes;
- ordenar cronológicamente;
- resolver ventana;
- respetar cutoff;
- comprobar precios;
- procesar una vez;
- aislar errores;
- continuar otras carteras/tareas;
- no enviar órdenes.

Integración esperada:

- daily-close o boundary canónico;
- coverageEndAt;
- ventana preopen;
- tareas existentes.

El fallo del recálculo no debe bloquear:

- registro post-trade;
- Book;
- cash;
- NAV;
- Track;
- Gate 2F;
- otras carteras.

==================================================
15. STARTUP CATCH-UP
==================================================

Crear o integrar:

catchUpGrowthCapitalRecalculations

Debe recuperar:

- servidor apagado en la ventana;
- PENDING;
- SCHEDULED vencido;
- WAITING_FOR_PRICES;
- FAILED_RETRYABLE;
- reinicio antes de COMMIT;
- reinicio después de COMMIT;
- revisión candidate abandonada;
- revisión canónica ya creada con request no completado.

Debe detectar resultados existentes y reparar metadatos cuando sea seguro.

No debe:

- volver a contabilizar aportaciones;
- crear dos revisiones;
- crear dos juegos de recomendaciones;
- superseder dos veces;
- enviar órdenes.

==================================================
16. IDEMPOTENCIA
==================================================

Claves separadas:

A. CONTRIBUTION IDEMPOTENCY

Ya implementada en 2A.

B. RECALCULATION IDEMPOTENCY

Derivada de:

- portfolio;
- contribution IDs ordenadas;
- input hash;
- applicable session;
- methodology/version.

Mismo input:

ALREADY_COMPLETED

Mismo key con inputs diferentes:

IDEMPOTENCY_CONFLICT

Retry después de COMMIT:

- devuelve la revisión existente;
- no duplica recomendaciones.

==================================================
17. CONCURRENCIA
==================================================

Probar:

- dos workers;
- scheduler + catch-up;
- dos requests para la misma aportación;
- dos aportaciones en la misma ventana;
- aporte confirmado durante RUNNING;
- formulario antiguo abierto;
- retry tras timeout;
- cancelación antes de RUNNING;
- intento de cancelación tras COMPLETED.

Usar:

BEGIN IMMEDIATE

o patrón transaccional canónico equivalente.

Solo un worker puede promover una revisión a CANONICAL.

==================================================
18. UI
==================================================

Extender la UI existente de Aportar capital.

No recuperar el panel eliminado.

Mostrar preview:

- importe;
- moneda;
- fecha/hora de disponibilidad;
- cash actual;
- cash proyectado;
- NAV actual;
- NAV proyectado;
- holdings sin cambios;
- mercado abierto/cerrado;
- apertura objetivo;
- cutoff;
- revisión vigente;
- recomendaciones pendientes afectadas.

Después de confirmar:

- contribution_id;
- movement_id;
- recalculation_request_id;
- cash posterior;
- NAV posterior;
- estado del recálculo;
- sesión objetivo.

Estados visibles:

- APORTACIÓN REGISTRADA;
- RECÁLCULO PROGRAMADO;
- ESPERANDO VENTANA;
- ESPERANDO PRECIOS;
- RECÁLCULO EN CURSO;
- NUEVA REVISIÓN DISPONIBLE;
- DIFERIDO POR CUTOFF;
- ERROR RECUPERABLE;
- ERROR FINAL.

En la revisión anterior:

“Revisión sustituida por aportación de capital.”

En la nueva:

- revisión anterior;
- aportaciones incluidas;
- capital anterior;
- total aportado;
- capital actualizado;
- cash;
- NAV;
- price session;
- applicable session;
- generated_at.

==================================================
19. ENDPOINTS
==================================================

Completar:

- GET contribution;
- list contributions;
- get recalculation request;
- list recalculation requests;
- get recalculation revision;
- get status;
- retry FAILED_RETRYABLE;
- cancel únicamente antes de RUNNING.

Conservar backward compatibility de:

POST /financial/:cartera/capital-contribution

Toda respuesta de confirmación debe incluir:

- contribution_id;
- movement_id;
- cash event;
- recalculation_request_id;
- target session;
- status.

No crear endpoint de trading.

==================================================
20. MIGRACIÓN v31
==================================================

Consolidar una única migración v30→v31.

No crear v32.

Antes de Producción:

- migrar base nueva;
- migrar copia exacta;
- probar migración repetida según contrato;
- integrity_check;
- foreign_key_check;
- preservar IDs;
- preservar hashes económicos;
- preservar movimientos;
- preservar holdings;
- preservar cash;
- preservar NAV;
- preservar revisiones;
- mantener tablas nuevas vacías.

La migración no debe:

- crear aportaciones;
- crear requests;
- crear revisiones;
- modificar Book;
- ejecutar recálculo.

==================================================
21. PRUEBAS ESPECÍFICAS
==================================================

Crear, como mínimo:

verify-capital-recalculation-schema.mjs
verify-capital-recalculation-session-resolution.mjs
verify-capital-recalculation-cutoff.mjs
verify-capital-recalculation-input-snapshot.mjs
verify-capital-recalculation-engine.mjs
verify-capital-recalculation-recommendations.mjs
verify-capital-recalculation-versioning.mjs
verify-capital-recalculation-supersession.mjs
verify-capital-recalculation-prices.mjs
verify-capital-recalculation-multiple-contributions.mjs
verify-capital-recalculation-idempotency.mjs
verify-capital-recalculation-concurrency.mjs
verify-capital-recalculation-scheduler.mjs
verify-capital-recalculation-catchup.mjs
verify-capital-recalculation-ui.mjs
verify-capital-recalculation-production-isolation.mjs

Casos obligatorios:

A. Fase 2A sigue verde.
B. Aportación no se contabiliza dos veces.
C. Cutoff está antes de apertura.
D. Mercado cerrado usa próxima ventana segura.
E. Mercado abierto usa siguiente sesión.
F. Premarket después del cutoff difiere.
G. Fin de semana correcto.
H. Festivo correcto.
I. Early close correcto.
J. Snapshot sellado.
K. Input hash estable.
L. Revisión candidate no sustituye la vigente.
M. Revisión validada se promueve atómicamente.
N. Fallo conserva revisión anterior.
O. Pendientes quedan superseded.
P. Fills Wio se preservan.
Q. Movimientos registrados se preservan.
R. Declinadas se conservan.
S. Solo recomendaciones normales.
T. Cero crp.
U. Recálculo desde estado completo.
V. Dos aportaciones se consolidan.
W. Doble worker no duplica.
X. Scheduler procesa una vez.
Y. Catch-up recupera.
Z. Falta de precio espera sin superseder.
AA. UI muestra estados.
AB. Migración no escribe economía.
AC. Despliegue no crea aportaciones/revisiones.
AD. Cero órdenes Wio.
AE. Gate 2F inactivo.

Requisitos para suites nuevas:

PASS > 0
FAIL = 0
NO_RESULT = 0

==================================================
22. REGRESIONES
==================================================

Ejecutar:

- verify-capital-contribution 29/0;
- verify-capital-contribution-http 16/0;
- verify-growth-remove-capital-resizing 11/0;
- growth-core-end-to-end 23/0;
- financial-core 31/0;
- migration;
- Book;
- Libro Mayor;
- holdings;
- cash;
- NAV;
- recommendation engine;
- calendar;
- scheduler;
- startup catch-up;
- repository integrity.

La comparación anterior detectó:

31 fallos preexistentes idénticos entre ed9282c y 1bf6a8b.

No considerar esto automáticamente suficiente para desplegar.

Clasificar esos fallos:

A. IRRELEVANT_PREEXISTING

No cubren código ni invariantes modificados.

B. RELEVANT_PREEXISTING

Cubren:

- aportaciones;
- cash;
- NAV;
- revisiones;
- precios;
- scheduler;
- calendario;
- migración;
- recommendation engine.

Cualquier RELEVANT_PREEXISTING debe:

- corregirse;
- o convertirse en fixture determinista correctamente aislado;
- quedar verde antes de desplegar.

No añadir skips.
No eliminar aserciones útiles.
No ocultar fallos.

Entregar la lista y clasificación de los 31.

==================================================
23. LAB
==================================================

Demostrar en LAB:

A. MERCADO CERRADO

- aportación confirmada;
- cash/NAV correctos;
- request programado;
- snapshot;
- revisión nueva;
- supersession.

B. MERCADO ABIERTO

- cash/NAV registrados;
- no revisión intradía;
- siguiente preopen.

C. CUTOFF

- aporte después del cutoff;
- siguiente ventana segura.

D. PRECIOS PENDIENTES

- WAITING_FOR_PRICES;
- revisión anterior vigente;
- retry exitoso.

E. REINICIO

- catch-up;
- una sola revisión.

F. DOS APORTACIONES

- dos movimientos;
- un snapshot consolidado;
- una revisión.

==================================================
24. PREVIEW PRODUCTIVO
==================================================

Antes del cutover:

- backup;
- SHA-256;
- abrir backup;
- integrity_check;
- foreign_key_check;
- baseline;
- migrar copia v30→v31;
- iniciar servidor sobre copia;
- ejecutar todas las suites;
- ejecutar scheduler;
- ejecutar catch-up;
- validar Chrome;
- comparar economía.

Comparar:

- Book;
- movimientos;
- holdings;
- cash;
- NAV;
- recomendaciones;
- revisiones;
- Wio fills;
- crp;
- Gate 2F.

No continuar si cambia cualquier dato económico por la migración o el
despliegue.

==================================================
25. DESPLIEGUE
==================================================

Desplegar únicamente con preview verde.

Orden:

1. detener escrituras;
2. una sola instancia;
3. backup productivo;
4. SHA-256;
5. integrity_check;
6. foreign_key_check;
7. baseline;
8. migrar v30→v31;
9. verificar schema;
10. desplegar backend;
11. desplegar UI;
12. reiniciar una sola instancia;
13. /ping;
14. env-info;
15. integrity_check;
16. foreign_key_check;
17. validar Book;
18. validar holdings;
19. validar cash;
20. validar NAV;
21. validar resizing ausente;
22. validar Aportar capital;
23. validar scheduler/catch-up inactivos sin requests;
24. validar Gate 2F inactivo.

El despliegue no puede crear:

- aportaciones;
- cash events;
- movimientos;
- recalculation requests;
- revisiones;
- recomendaciones;
- supersession;
- órdenes Wio.

==================================================
26. CHROME
==================================================

Validar en Producción sin confirmar una aportación real de prueba:

- aplicación carga;
- errores JS de Mizan = 0;
- resizing ausente;
- crp ausentes de operación;
- Aportar capital presente;
- preview accesible sin escritura;
- cash/NAV proyectados;
- estado de mercado;
- sesión objetivo;
- cutoff visible;
- capital objetivo no crea cash.

Validar confirmación y recálculo completo únicamente en LAB/copia.

No contar errores de extensiones de Chrome como errores de Mizan, pero
documentarlos por origen.

==================================================
27. SEGURIDAD
==================================================

Después del despliegue:

PRODUCTION_CONTRIBUTIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_CASH_EVENTS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_RECALCULATION_REQUESTS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_REVISIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_RECOMMENDATIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_SUPERSESSIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_BROKER_ORDERS_SENT =
0

GATE_2F_REMAINS_INACTIVE =
PASS

==================================================
28. COMMIT
==================================================

Crear un único commit local sobre 1bf6a8b:

feat(growth): complete capital contribution recalculation engine

No hacer push.
No crear tag.

Ejecutar secret scan.

==================================================
29. ENTREGA FINAL
==================================================

CANONICAL_HEAD_BEFORE =
1bf6a8b

CANONICAL_HEAD_AFTER =
<hash>

BRANCH =
release/defensiva-operational-parity-v1.1

PRODUCTION_SCHEMA_BEFORE =
v30

PRODUCTION_SCHEMA_AFTER =
v31

V31_MIGRATION =
PASS/FAIL

V31_INTEGRITY =
PASS/FAIL

V31_FOREIGN_KEYS =
PASS/FAIL

PHASE_2A_REGRESSION =
PASS/FAIL

ONE_CONTRIBUTION_ONE_CASH_EFFECT =
PASS/FAIL

ONE_CONTRIBUTION_ONE_RECALCULATION_REQUEST =
PASS/FAIL

PREOPEN_CUTOFF_BEFORE_MARKET_OPEN =
PASS/FAIL

PREOPEN_CUTOFF_MARGIN_MINUTES =
<número>

NYSE_SESSION_RESOLUTION =
PASS/FAIL

CANONICAL_INPUT_SNAPSHOT =
PASS/FAIL

INPUT_HASH_STABLE =
PASS/FAIL

RECALCULATION_FROM_CANONICAL_STATE =
PASS/FAIL

ONLY_NORMAL_RECOMMENDATIONS =
PASS/FAIL

GENERIC_RESIZING_RECOMMENDATIONS_CREATED =
0

REVISION_VERSIONING =
PASS/FAIL

ATOMIC_SUPERSESSION =
PASS/FAIL

FAILED_RECALCULATION_PRESERVES_CURRENT_REVIEW =
PASS/FAIL

EXECUTED_WIO_FILLS_PRESERVED =
PASS/FAIL

RECORDED_MOVEMENTS_PRESERVED =
PASS/FAIL

DECLINED_RECOMMENDATIONS_PRESERVED =
PASS/FAIL

MULTIPLE_CONTRIBUTION_CONSOLIDATION =
PASS/FAIL

CONTRIBUTION_IDEMPOTENCY =
PASS/FAIL

RECALCULATION_IDEMPOTENCY =
PASS/FAIL

RECALCULATION_CONCURRENCY =
PASS/FAIL

WAITING_FOR_PRICES =
PASS/FAIL

SCHEDULER_INTEGRATION =
PASS/FAIL

STARTUP_CATCHUP =
PASS/FAIL

CHROME_GREEN =
PASS/FAIL

MIZAN_JAVASCRIPT_ERRORS =
<número>

PREEXISTING_FAILURES_TOTAL =
31

PREEXISTING_FAILURES_IRRELEVANT =
<número>

PREEXISTING_FAILURES_RELEVANT_FIXED =
<número>

PREEXISTING_FAILURES_RELEVANT_REMAINING =
0

PRODUCTION_CONTRIBUTIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_CASH_EVENTS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_RECALCULATION_REQUESTS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_REVISIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_RECOMMENDATIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_SUPERSESSIONS_CREATED_BY_DEPLOYMENT =
0

PRODUCTION_BROKER_ORDERS_SENT =
0

GENERIC_CAPITAL_RESIZING_PROPOSAL_REMOVED =
PASS

NEW_CRP_RECOMMENDATIONS_POSSIBLE =
NO

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

Detenerse después de completar, desplegar y verificar Fase 2B.
No crear una aportación productiva ficticia.