CONTINUACIÓN OBLIGATORIA · APORTACIONES Y ELIMINACIÓN DEL RESIZING

La auditoría y reclasificación de las tres decisiones crp queda aprobada:

CANONICAL_HEAD =
ce510c5

CRP_AUDITED =
3/3

CRP_REAL_WIO_EXECUTIONS =
0

CRP_NON_ECONOMIC_DECISIONS =
3

CRP_REMOVED_FROM_WIO_REENTRY_QUEUE =
3

WIO_REENTRY_RECOMMENDATIONS_REMAINING =
18

La cola correcta de fills Wio pendientes es 18, no 21.

==================================================
1. CORRECCIÓN DE INTERPRETACIÓN
==================================================

Los 18 fills pendientes y la falta de conciliación final impiden:

- ejecutar una aportación real en Producción;
- generar una nueva revisión productiva;
- superseder recomendaciones productivas;
- utilizar el cash actual como base económica definitiva.

Pero no impiden:

- eliminar completamente el panel abstracto de resizing;
- eliminar la generación futura de propuestas crp;
- construir el registro contable de aportaciones;
- implementar el motor de recalibración;
- implementar el calendario y scheduler;
- implementar versionado y supersession;
- probar todo en LAB y sobre copias;
- desplegar la infraestructura desactivada;
- añadir un gate que impida usarla hasta reconciliar la cartera.

La instrucción:

“No generar una nueva revisión productiva hasta que exista una aportación real
confirmada”

no significa:

“No construir ni desplegar el motor”.

Por tanto, continuar con el gate completo, pero mantener bloqueada toda
ejecución económica productiva.

==================================================
2. ESTADO PRODUCTIVO OBLIGATORIO
==================================================

Mientras existan los 18 fills pendientes:

GROWTH_BOOK_RECONCILIATION_STATUS =
PENDING_WIO_FILLS

CAPITAL_CONTRIBUTION_RECALCULATION_ENABLED =
FALSE

PRODUCTION_CONTRIBUTION_ALLOWED =
FALSE

PRODUCTION_REVIEW_RECALCULATION_ALLOWED =
FALSE

El sistema debe devolver:

BOOK_RECONCILIATION_REQUIRED

si se intenta confirmar una aportación o ejecutar una recalibración productiva
antes de cerrar los fills.

No crear:

- aportaciones productivas;
- movimientos productivos de aportación;
- nuevas revisiones productivas;
- supersession productiva;
- recomendaciones productivas nuevas.

==================================================
3. ELIMINAR COMPLETAMENTE EL PANEL
==================================================

Completar ahora:

GENERIC_CAPITAL_RESIZING_PROPOSAL_REMOVED =
PASS

Eliminar de la UI operativa:

- “Propuesta de resizing por capital”;
- botones Aceptar/Declinar de crp;
- contadores operativos de crp;
- acciones de reentrada Wio de crp;
- mensajes de Libro Mayor asociados a crp;
- cualquier generación futura de capital_resizing_line.

Conservar las tres decisiones históricas como auditoría:

LEGACY_NON_ECONOMIC_CAPITAL_RESIZING_DECISION

No borrarlas.

No mostrarlas en:

- recomendaciones operativas;
- cola de fills;
- Libro Mayor;
- movimientos pendientes;
- contadores de operaciones.

La retirada completa del panel no depende de los 18 fills.

==================================================
4. CONSTRUIR EL REGISTRO DE APORTACIONES
==================================================

Implementar ahora el flujo:

REGISTRAR APORTACIÓN DE CAPITAL

Estados:

- DRAFT;
- PENDING_AVAILABILITY;
- CONFIRMED_AVAILABLE;
- CANCELLED;
- REJECTED.

Campos:

- cuenta/cartera;
- importe;
- moneda;
- available_at;
- referencia Wio opcional;
- comentario opcional;
- idempotency key.

Sin embargo, en Producción, mientras el Book no esté reconciliado:

- permitir abrir preview;
- no permitir confirmar;
- mostrar BOOK_RECONCILIATION_REQUIRED;
- no escribir movimiento;
- no aumentar cash;
- no aumentar NAV.

En LAB y tests sí debe poder confirmarse usando fixtures reconciliados.

==================================================
5. MOTOR ECONÓMICO
==================================================

Implementar y probar en LAB:

confirmed contribution
→ movimiento EXTERNAL_CAPITAL_CONTRIBUTION
→ aumento de cash
→ aumento de NAV
→ rentabilidad neutral
→ solicitud de recalibración
→ supersession de pendientes
→ nueva revisión desde cero

Debe cumplirse:

cash_after =
cash_before + contribution_amount

NAV_after =
NAV_before + contribution_amount

return_created =
0

No modificar holdings.

No crear compras ni ventas.

No enviar órdenes a Wio.

No utilizar el capital objetivo como cash.

==================================================
6. MOTOR DE RECALIBRACIÓN
==================================================

Implementar:

recalculateGrowthRecommendationsAfterCapitalContribution

Debe recalcular desde:

- holdings reales sellados;
- cash reconciliado;
- aportaciones CONFIRMED_AVAILABLE;
- NAV;
- política vigente;
- precios canónicos;
- sesión aplicable.

No usar:

- recomendaciones anteriores como base incremental;
- capital objetivo sin cash;
- resizing aceptado;
- fills pendientes sin registrar;
- proceeds estimados.

Generar recomendaciones normales:

- MANTENER;
- INCORPORAR;
- AUMENTAR;
- REDUCIR;
- ELIMINAR.

No generar:

CAPITAL_RESIZING

==================================================
7. GATE DE RECONCILIACIÓN
==================================================

Crear una función canónica:

isGrowthCapitalRecalculationAllowed

Debe devolver true únicamente si:

- Book reconciliado;
- Libro Mayor reconciliado;
- holdings reconciliados;
- no existen fills Wio completos pendientes de registro;
- cash base aceptado/reconciliado;
- aportación CONFIRMED_AVAILABLE;
- una sola instancia operativa;
- inputs sellados.

Con el estado productivo actual debe devolver:

false

Motivo:

18 WIO FILLS PENDING REGISTRATION

El preview debe indicar exactamente los tickers pendientes.

No acoplar la implementación al número 18.

Calcularlo dinámicamente.

==================================================
8. REGLA TEMPORAL
==================================================

Usar America/New_York y calendario bursátil canónico.

Aportación confirmada con mercado cerrado:

- objetivo = próxima apertura;
- recalcular antes de esa apertura si supera el cutoff de seguridad.

Aportación confirmada con mercado abierto:

- registrar aportación;
- no recalibrar a mitad de sesión;
- recalcular antes de la apertura de la siguiente sesión.

Aportación confirmada demasiado cerca del cutoff:

RECALCULATION_DEFERRED_BY_CUTOFF

No generar una revisión parcial.

No usar horas hardcodeadas sin calendario.

==================================================
9. VERSIONADO Y SUPERSESSION
==================================================

Implementar en LAB:

- capital_basis;
- cash_basis;
- NAV_basis;
- holdings_snapshot_id;
- contribution_ids;
- applicable_market_session;
- generated_at;
- price basis;
- input hash;
- supersedes_review_run_id;
- superseded_by_review_run_id;
- supersession_reason.

Razón:

CAPITAL_CONTRIBUTION_CONFIRMED

Reglas:

PENDING_NOT_EXECUTED
→ SUPERSEDED_BY_CAPITAL_CHANGE

EXECUTED_IN_WIO_PENDING_REGISTRATION
→ conservar sin cambios

ACCEPTED_AND_RECORDED
→ conservar sin cambios

DECLINED
→ conservar como historia

No aplicar estas transiciones a datos productivos mientras el gate de
reconciliación esté cerrado.

==================================================
10. SCHEMA
==================================================

Auditar si v30 soporta:

- aportaciones reales;
- disponibilidad;
- idempotencia;
- contribution lineage;
- recalculation request;
- review supersession;
- cutoff/sesión objetivo;
- reconciliation gate.

Si v30 ya lo soporta:

- no migrar.

Si falta soporte:

- crear NEXT_AVAILABLE_SCHEMA_VERSION;
- migración aditiva mínima;
- probar únicamente en copia antes del despliegue;
- no insertar aportaciones productivas.

No reutilizar las tablas crp con una semántica económica distinta.

==================================================
11. UI
==================================================

Eliminar el panel de resizing ahora.

Añadir:

REGISTRAR APORTACIÓN DE CAPITAL

Mientras existan fills pendientes, la pantalla debe mostrar:

“Registro temporalmente bloqueado: existen ejecuciones Wio pendientes de
incorporar al Libro Mayor. Complete la conciliación antes de confirmar una
aportación.”

Puede permitir introducir datos para preview, pero no confirmar ni escribir.

Mostrar:

- importe;
- moneda;
- disponibilidad;
- cash actual;
- cash proyectado;
- NAV actual;
- NAV proyectado;
- sesión objetivo;
- estado de reconciliación;
- fills pendientes.

No confundir una aportación con un cambio de capital objetivo.

==================================================
12. PRUEBAS
==================================================

Construir y ejecutar en LAB:

- aportación aumenta cash;
- aportación aumenta NAV;
- rentabilidad neutral;
- aportación no modifica holdings;
- cambio de capital objetivo no crea cash;
- mercado cerrado recalcula para próxima apertura;
- mercado abierto recalcula para siguiente sesión;
- cutoff difiere;
- pendientes quedan superseded;
- fill Wio ejecutado permanece intacto;
- recomendación registrada permanece intacta;
- no aparece resizing;
- no se generan crp nuevas;
- doble aportación es idempotente;
- doble recálculo no duplica revisión;
- gate productivo cerrado por fills pendientes;
- intento productivo no escribe;
- cero órdenes Wio.

Corregir los seis fallos del fixture capital-resizing-phase2 si el problema es
aislamiento del fixture.

No modificar Producción para satisfacer tests.

==================================================
13. DESPLIEGUE PERMITIDO
==================================================

Puede desplegarse:

- eliminación del panel;
- nueva UI de aportación en estado bloqueado;
- servicios;
- schema, si fuera necesario;
- scheduler;
- versionado;
- policy gate;
- pruebas.

No puede desplegarse activado:

- confirmación productiva de aportaciones;
- recálculo productivo;
- supersession productiva.

Después del despliegue:

GENERIC_CAPITAL_RESIZING_PROPOSAL_REMOVED =
PASS

CAPITAL_CONTRIBUTION_ENGINE_DEPLOYED =
PASS

CAPITAL_CONTRIBUTION_ENGINE_ENABLED =
FALSE

RECALCULATION_BLOCK_REASON =
PENDING_WIO_FILL_REGISTRATION

PRODUCTION_CONTRIBUTIONS_CREATED =
0

PRODUCTION_REVIEWS_RECALCULATED =
0

==================================================
14. ACTIVACIÓN POSTERIOR
==================================================

Después de que Omar introduzca los 18 fills:

1. verificar movimientos;
2. verificar holdings;
3. verificar comisiones;
4. verificar cash;
5. recibir o aceptar el cash base;
6. cerrar conciliación;
7. ejecutar Chrome A–H;
8. volver a evaluar isGrowthCapitalRecalculationAllowed;
9. habilitar confirmación de aportaciones;
10. no crear aportación hasta que Omar introduzca una real.

La habilitación no debe crear automáticamente:

- aportación;
- revisión;
- recomendaciones;
- operaciones.

==================================================
15. COMMIT
==================================================

Crear un único commit local:

fix(growth): replace capital resizing with guarded contribution recalculation

No hacer push.
No crear tag.

==================================================
16. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE =
ce510c5

CANONICAL_HEAD_AFTER =
<hash>

SCHEMA_BEFORE =
v30

SCHEMA_AFTER =
<v30/NEXT_AVAILABLE_SCHEMA_VERSION>

GENERIC_CAPITAL_RESIZING_PROPOSAL_REMOVED =
PASS/FAIL

LEGACY_CRP_DECISIONS_PRESERVED =
3/3

NEW_CRP_RECOMMENDATIONS_POSSIBLE =
NO

CAPITAL_CONTRIBUTION_ENGINE =
PASS/FAIL

CAPITAL_CONTRIBUTION_ENGINE_DEPLOYED =
PASS/FAIL

CAPITAL_CONTRIBUTION_ENGINE_ENABLED =
FALSE

RECALCULATION_GATE =
PASS/FAIL

RECALCULATION_BLOCK_REASON =
PENDING_WIO_FILL_REGISTRATION

WIO_REENTRY_RECOMMENDATIONS_REMAINING =
<número dinámico>

CAPITAL_TARGET_CHANGE_CREATES_CASH =
NO

CONFIRMED_CONTRIBUTION_CREATES_CASH_LAB =
PASS/FAIL

CONTRIBUTION_INCREASES_NAV_LAB =
PASS/FAIL

CONTRIBUTION_RETURN_NEUTRAL_LAB =
PASS/FAIL

MARKET_CLOSED_RECALCULATION_LAB =
PASS/FAIL

MARKET_OPEN_NEXT_SESSION_RECALCULATION_LAB =
PASS/FAIL

PREOPEN_CUTOFF_LAB =
PASS/FAIL

REVIEW_SUPERSESSION_LAB =
PASS/FAIL

EXECUTED_WIO_FILLS_PRESERVED =
PASS/FAIL

GENERIC_RESIZING_UI_REMOVED =
PASS/FAIL

CONTRIBUTION_UI_GUARDED =
PASS/FAIL

CAPITAL_RESIZING_TESTS =
PASS/FAIL

PRODUCTION_CONTRIBUTIONS_CREATED =
0

PRODUCTION_REVIEWS_RECALCULATED =
0

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

Detenerse después de desplegar el motor desactivado y eliminar completamente
el resizing abstracto.