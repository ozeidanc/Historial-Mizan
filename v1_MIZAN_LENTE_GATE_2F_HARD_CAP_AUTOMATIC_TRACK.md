MIZAN · LA LENTE
GATE 2F · ASIGNACIÓN PROSPECTIVA Y ACTUALIZACIÓN AUTOMÁTICA DEL TRACK PAPEL
TESIS → CARTERA EXISTENTE/NUEVA → HARD CAP → COMPOSICIÓN EOD → TRACK

NATURALEZA

Esta ejecución autoriza:

1. conectar una tesis aprobada con el flujo de asignación papel;
2. permitir seleccionar una cartera catalizada existente;
3. permitir crear una cartera catalizada nueva;
4. implementar MANUAL_NOTIONAL_HARD_CAP;
5. preservar requested_notional;
6. comprobar capacidad y Paper Cash;
7. crear solicitudes pendientes cuando no exista capacidad;
8. permitir asignaciones parciales solo mediante confirmación explícita;
9. permitir aportaciones externas papel explícitas;
10. crear composiciones prospectivas point-in-time;
11. ejecutar compras internas al cierre oficial;
12. actualizar automáticamente el Track Papel;
13. iniciar automáticamente el Track de una cartera nueva;
14. actualizar el selector de carteras de forma dinámica;
15. activar la política prospectiva desde una sesión futura segura;
16. desplegar y validar el flujo completo;
17. crear un único commit local.

No autoriza:

- modificar el histórico anterior a la activación;
- reinterpretar composiciones legacy;
- modificar requested_notional históricos;
- certificar los 36 objetos históricos inferidos;
- rebalancear automáticamente posiciones existentes;
- normalizar automáticamente a capital/N;
- realizar aportaciones implícitas;
- permitir cash negativo;
- permitir leverage;
- ejecutar operaciones reales;
- conectar con Wio;
- modificar Book, cash o NAV reales;
- modificar holdings legacy;
- modificar snapshots legacy;
- modificar cat_composicion_log;
- modificar tesis históricas;
- modificar catalizadores;
- modificar Campo de Caza;
- modificar el motor histórico AS_OPERATED;
- modificar snapshots históricos ya certificados;
- hacer push;
- crear tag;
- realizar operaciones Git remotas.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = 0466595
schema esperado = v29
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Estado confirmado:

PAPER_PORTFOLIOS = 5
PAPER_GLOBAL_POSITIONS = 82
PAPER_MEMBERSHIP_EPISODES = 121
PAPER_TRACK_SNAPSHOTS = 54

CANONICAL_ECONOMIC_RECONSTRUCTION_VERSION = 1

PAPER_TRACK_METHOD =
PRICE_RETURN_TWR

HISTORICAL_TRACK_POLICY =
AS_OPERATED

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

PROSPECTIVE_ALLOCATION_POLICY =
MANUAL_NOTIONAL_HARD_CAP

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

PROSPECTIVE_AUTO_REBALANCE =
DISABLED

PARTIAL_ALLOCATION_POLICY =
REQUIRE_EXPLICIT_CONFIRMATION

LEVERAGE_POLICY =
DISABLED

EXIT_POLICY =
KEEP_AS_PAPER_CASH

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

COMMISSION_POLICY =
NOT_MODELED

BENCHMARK =
SPY

BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

Vista operativa:

LensPaperTrackView

Endpoint del Track:

GET /lens/paper-track/:portfolioId

La gráfica y el motor diario ya están terminados.

Gate 2F no debe reconstruirlos. Solo debe alimentar prospectivamente sus
fuentes canónicas.

==================================================
1. RESULTADO FUNCIONAL
==================================================

Después de Gate 2F, el flujo debe ser:

TESIS APROBADA
→ PROPUESTA DE ASIGNACIÓN PAPEL
→ SELECCIÓN DE CARTERA EXISTENTE O NUEVA
→ REQUESTED_NOTIONAL
→ PREVIEW DE CAPACIDAD
→ CONFIRMACIÓN EXPLÍCITA
→ RESERVA DE CAPACIDAD
→ ESPERA DEL CIERRE EFECTIVO
→ PRECIO EOD VALIDADO
→ COMPRA INTERNA
→ NUEVA COMPOSICIÓN EOD
→ SNAPSHOT DEL TRACK
→ GRÁFICA ACTUALIZADA

La actualización del Track debe ser automática después de la confirmación y
del cierre oficial aplicable.

No debe requerir:

- edición manual de base de datos;
- ejecución manual de backfill;
- recarga administrativa;
- intervención en la gráfica;
- reaplicar una migración;
- modificar holdings legacy.

==================================================
2. PRINCIPIO DE CONFIRMACIÓN
==================================================

Una tesis aprobada no crea automáticamente exposición económica.

Una tesis aprobada puede crear:

allocation_request.status =
DRAFT

o:

AWAITING_CONFIRMATION

No puede crear todavía:

- global position activa;
- membership efectivo;
- composición económica;
- transacción;
- exposición;
- snapshot con la posición;
- retorno;
- benchmark flow.

La incorporación económica solo puede comenzar después de una confirmación
explícita de Omar.

La confirmación debe incluir:

- tesis;
- ticker;
- cartera;
- requested_notional;
- allocated_notional propuesto;
- Paper Cash disponible;
- Paper Cash reservado;
- Paper Cash restante;
- política;
- fecha/sesión efectiva estimada;
- warnings;
- acción seleccionada.

No considerar confirmación:

- abrir el modal;
- seleccionar una cartera;
- escribir un importe;
- cerrar el modal;
- aprobar la tesis;
- recargar la página.

==================================================
3. FLUJO DESDE LA TESIS
==================================================

Cuando una tesis alcance el estado que actualmente permite pasarla a una
cartera catalizada, mostrar una acción explícita:

AÑADIR A CARTERA PAPEL

La acción debe ofrecer:

A. CARTERA EXISTENTE

- listar las carteras catalizadas disponibles;
- mostrar capital configurado;
- Paper NAV;
- Paper Cash;
- cash reservado;
- capacidad disponible;
- posiciones activas;
- solicitudes pendientes;
- política efectiva.

B. CARTERA NUEVA

- nombre;
- slug;
- descripción opcional;
- moneda;
- capital inicial;
- requested_notional de la primera posición;
- benchmark;
- metodología;
- preview;
- confirmación.

No seleccionar automáticamente una cartera sin intervención del usuario.

No crear automáticamente una cartera solo porque todas las existentes estén
sin capacidad.

==================================================
4. DEFINICIÓN DE CAPACIDAD
==================================================

Por cartera:

gross_paper_cash =
Paper Cash del último estado económico certificado
+ flujos externos confirmados y efectivos
- retiradas externas confirmadas y efectivas

reserved_cash =
suma de asignaciones confirmadas pendientes de ejecución
que reservan capacidad en esa cartera.

available_capacity =
max(0, gross_paper_cash - reserved_cash)

La comprobación de capacidad debe ser transaccional.

No utilizar:

- capital configurado menos suma de requested_notional históricos;
- holdings.importe;
- allocated_notional legacy;
- composición actual aplicada hacia atrás;
- NAV real;
- cash real.

Para solicitudes simultáneas:

- serializar la comprobación;
- bloquear o usar una transacción SQLite;
- respetar confirmed_at;
- usar id/idempotency key como desempate;
- impedir doble reserva del mismo cash.

Declarar:

CAPACITY_RACE_PROTECTION = PASS/FAIL

==================================================
5. MANUAL_NOTIONAL_HARD_CAP
==================================================

Si:

available_capacity >= requested_notional

entonces:

allocated_notional =
requested_notional

allocation_status =
RESERVED_FOR_EXECUTION

Después del cierre y compra:

allocation_status =
FULLY_ALLOCATED

Si:

available_capacity < requested_notional

entonces, por defecto:

allocated_notional =
0

allocation_status =
PENDING_CAPITAL

No realizar automáticamente una asignación parcial.

No modificar posiciones existentes.

No normalizar la cartera.

No aumentar el capital.

No crear cash negativo.

No utilizar leverage.

No cambiar pesos existentes hasta que se ejecute una orden explícita que los
afecte.

==================================================
6. ASIGNACIÓN PARCIAL
==================================================

Si:

0 < available_capacity < requested_notional

la UI puede ofrecer:

ASIGNAR PARCIALMENTE

Debe mostrar:

- requested_notional;
- available_capacity;
- proposed_allocated_notional;
- diferencia pendiente;
- peso resultante;
- Paper Cash posterior;
- sesión efectiva;
- warning de asignación parcial.

Solo ejecutar después de una segunda confirmación explícita.

Resultado:

allocation_status =
PARTIALLY_ALLOCATED

requested_notional conserva el importe original.

allocated_notional conserva únicamente el importe confirmado.

La diferencia:

unallocated_notional =
requested_notional - allocated_notional

debe permanecer visible.

No crear automáticamente otra solicitud por la diferencia.

No incrementar después la posición automáticamente si aparece cash nuevo.

Cualquier ampliación posterior requiere una nueva confirmación.

==================================================
7. SOLICITUDES PENDIENTES
==================================================

Estados mínimos:

- DRAFT;
- AWAITING_CONFIRMATION;
- PENDING_CAPITAL;
- PENDING_PARTIAL_CONFIRMATION;
- PENDING_EXTERNAL_CONTRIBUTION;
- PENDING_REBALANCE_CONFIRMATION;
- RESERVED_FOR_EXECUTION;
- PENDING_MARKET_CLOSE;
- PENDING_PRICE;
- FULLY_ALLOCATED;
- PARTIALLY_ALLOCATED;
- CANCELLED;
- REJECTED;
- SUPERSEDED;
- FAILED.

Una solicitud pendiente:

- conserva requested_notional;
- no crea exposición;
- no crea market value;
- no crea membership económico efectivo;
- no crea transacción;
- no modifica el Track;
- no modifica SPY;
- no consume retorno;
- no cambia Paper Unit Value.

Una solicitud confirmada puede reservar cash antes de su ejecución.

Una solicitud cancelada debe liberar la reserva de forma atómica.

Una solicitud fallida debe:

- liberar la reserva cuando ya no sea ejecutable;
- conservar el motivo;
- no dejar capacidad bloqueada indefinidamente.

==================================================
8. FECHA Y SESIÓN EFECTIVA
==================================================

Para operaciones prospectivas no utilizar cierres anteriores a la
confirmación.

Resolver con America/New_York y el calendario NYSE.

A. CONFIRMACIÓN ANTES DE LA APERTURA

La operación será candidata al cierre oficial de esa sesión.

B. CONFIRMACIÓN DURANTE MERCADO ABIERTO

La operación será candidata al cierre oficial de esa sesión.

C. CONFIRMACIÓN DESPUÉS DEL CIERRE

La operación será candidata al cierre de la siguiente sesión bursátil.

D. FIN DE SEMANA O FESTIVO

La operación será candidata al cierre de la siguiente sesión bursátil.

E. EARLY CLOSE

Usar la hora específica de cierre de esa sesión.

Una confirmación posterior al early close pasa a la siguiente sesión.

No utilizar:

- cierre anterior a confirmed_at;
- precio live;
- apertura;
- previous close;
- precio de entrada legacy;
- interpolación.

Estados temporales:

RESERVED_FOR_EXECUTION
→ PENDING_MARKET_CLOSE
→ PENDING_PRICE
→ FULLY_ALLOCATED/PARTIALLY_ALLOCATED

==================================================
9. PRECIO Y EJECUCIÓN
==================================================

La incorporación se ejecuta usando:

PRICE_BASIS =
OFFICIAL_CLOSE_NOMINAL

Proveedor:

el proveedor canónico y el manifiesto versionado del Track Papel.

Después de cerrar la sesión:

1. obtener el cierre EOD;
2. validar ticker y sesión;
3. extender el manifiesto de precios;
4. persistir evidencia;
5. comprobar nuevamente la reserva;
6. crear posición/membership si corresponde;
7. crear composición EOD;
8. crear transacción económica;
9. crear snapshot;
10. actualizar el endpoint;
11. actualizar la gráfica.

Si falta el precio:

allocation_status =
PENDING_PRICE

No usar fallback.

No crear la compra.

No liberar la reserva automáticamente mientras la operación siga siendo
válida y reintentable.

El catch-up debe reintentarla.

==================================================
10. NEUTRALIDAD ECONÓMICA DE LA COMPRA
==================================================

Para una nueva incorporación:

NAV_PRE_TRADE =
Paper Cash antes
+ valor de mercado de posiciones existentes al cierre.

La compra:

- reduce Paper Cash;
- aumenta Paper Market Value;
- no cambia Paper NAV;
- no cambia Paper Units;
- no cambia Paper Unit Value;
- no crea retorno por sí misma.

Debe cumplirse:

NAV_POST_TRADE = NAV_PRE_TRADE

units_after = units_before

unit_value_after = unit_value_before

internal_transaction_return = 0

La primera variación de precio de la nueva posición se reconoce desde la
sesión siguiente.

==================================================
11. CARTERA EXISTENTE
==================================================

Al incorporar una posición a una cartera existente:

- conservar todas las posiciones existentes;
- conservar sus cantidades;
- conservar su valor económico;
- no rebalancearlas;
- no aplicar equal-weight;
- no aplicar pro-rata;
- no alterar sus requested_notional;
- no cambiar anchors;
- no reescribir snapshots anteriores.

Crear:

- allocation request;
- global position, si no existe;
- membership episode;
- nueva composition version;
- composition member;
- transacción PURCHASE;
- snapshot de la sesión;
- referencias de precio y confirmación.

La nueva composition version debe contener:

- posiciones anteriores sin cambios de cantidad;
- nueva posición con su cantidad comprada;
- Paper Cash resultante;
- pesos observados después de la compra.

Los pesos son consecuencia de valores de mercado.

No son un objetivo de rebalanceo automático.

==================================================
12. POSICIÓN 11 Y EXCESO DE CAPACIDAD
==================================================

Ejemplo:

configured_capital =
10.000

10 posiciones existentes asignadas por 1.000

Paper Cash =
0

Nueva solicitud:

requested_notional =
1.000

Resultado:

allocated_notional =
0

allocation_status =
PENDING_CAPITAL

La posición 11:

- no entra en la composición;
- no crea membership económico;
- no crea compra;
- no diluye las diez posiciones;
- no cambia el Track;
- no aumenta el capital;
- no crea cash negativo.

Debe permanecer pendiente hasta que Omar elija explícitamente:

- aportar capital;
- vender una posición;
- reducir otra posición;
- confirmar asignación parcial si aparece cash;
- cancelar la solicitud.

Declarar:

POSITION_11_PENDING = PASS/FAIL

==================================================
13. APORTACIÓN EXTERNA EXPLÍCITA
==================================================

Desde una solicitud PENDING_CAPITAL, ofrecer:

APORTAR CAPITAL PAPEL

La aportación debe requerir:

- importe;
- cartera;
- preview;
- efecto sobre Paper Cash;
- unidades que se emitirán;
- sesión efectiva;
- confirmación.

Una aportación externa:

- aumenta Paper NAV;
- aumenta Paper Cash;
- emite Paper Units;
- no cambia Paper Unit Value;
- se replica en SPY;
- queda registrada en paper_portfolio_flow.

Fórmula:

units_delta =
external_contribution
/
pre_flow_unit_value

Después de la aportación puede ejecutarse la asignación reservada, si Omar lo
confirma dentro del mismo flujo.

No aportar automáticamente el importe faltante.

No utilizar capital real.

No conectar con una cuenta real.

==================================================
14. REBALANCEO EXPLÍCITO
==================================================

Desde PENDING_CAPITAL puede ofrecerse:

PROPONER REBALANCEO

El rebalanceo no se ejecuta en este Gate salvo que el flujo completo quede
implementado con:

- posiciones a reducir;
- importes;
- cantidades;
- cierres;
- turnover;
- Paper Cash resultante;
- nueva posición;
- NAV continuity;
- preview;
- confirmación explícita.

Nunca usar:

- equal-weight automático;
- capital/N;
- normalización general;
- venta automática de todas las posiciones.

Si el rebalanceo explícito no queda implementado completamente:

- mantener la opción deshabilitada;
- mostrar “Próximamente” o equivalente;
- no simular que se ejecutó.

La ausencia de rebalanceo explícito no bloquea el hard cap.

==================================================
15. CANCELACIÓN Y REVOCACIÓN
==================================================

Antes de la ejecución al cierre, Omar puede cancelar una solicitud.

La cancelación:

- libera la reserva;
- conserva auditoría;
- no crea posición;
- no crea membership económico;
- no crea transacción;
- no cambia el Track.

Si el cierre ya fue procesado y la compra fue ejecutada:

- no cancelar retroactivamente;
- ofrecer una salida papel explícita;
- la salida será una nueva transacción.

No borrar historia para simular una cancelación tardía.

==================================================
16. SALIDA DE UNA POSICIÓN
==================================================

Para mantener operativa la política:

EXIT_POLICY =
KEEP_AS_PAPER_CASH

Permitir una salida papel explícita de una posición activa.

Debe requerir:

- cartera;
- ticker;
- cantidad o salida total;
- preview;
- cierre efectivo;
- confirmación.

Al cierre:

- vender usando cierre oficial;
- reducir o cerrar membership;
- aumentar Paper Cash;
- conservar Paper Units;
- conservar Paper Unit Value neutral a la operación;
- crear nueva composición;
- actualizar snapshot;
- no retirar capital automáticamente.

Una salida total:

- cierra el membership;
- no elimina la global position histórica;
- mantiene auditoría.

No convertir automáticamente el Paper Cash resultante en otra posición.

==================================================
17. REASIGNACIÓN ENTRE CARTERAS
==================================================

Una reasignación prospectiva debe ser explícita y atómica.

Antes de confirmar:

ORIGEN

- posición;
- cantidad;
- valor;
- Paper Cash posterior;
- composición posterior.

DESTINO

- requested_notional;
- available_capacity;
- Paper Cash;
- estado hard cap;
- composición propuesta.

Regla predeterminada:

Si la cartera destino no tiene capacidad suficiente:

- no cerrar la posición en origen;
- marcar la propuesta PENDING_CAPITAL;
- no ejecutar parcialmente sin confirmación;
- no duplicar la posición.

Si ambas patas pueden ejecutarse:

- mismo reassignment_group_id;
- misma sesión efectiva;
- venta en origen;
- compra en destino;
- cero flujo externo;
- unidades independientes por cartera;
- Track independiente por cartera.

No transferir Paper Units entre carteras.

==================================================
18. CREACIÓN DE CARTERA NUEVA
==================================================

Permitir crear una cartera catalizada nueva desde el flujo de asignación.

Campos obligatorios:

- nombre;
- slug único;
- capital inicial;
- moneda;
- benchmark = SPY;
- metodología = PAPER_PRICE_RETURN_TWR_V1;
- política = MANUAL_NOTIONAL_HARD_CAP;
- requested_notional de la primera posición;
- confirmación.

Validaciones:

- nombre válido;
- slug único;
- capital > 0;
- moneda soportada;
- requested_notional > 0;
- no duplicar idempotency key;
- no duplicar cartera por doble clic.

La creación debe preparar:

- cartera;
- paper_portfolio_track_config;
- política activa para esa cartera;
- PAPER_INITIAL_CONTRIBUTION;
- Paper Unit Value inicial = 100;
- Paper Units iniciales;
- Paper Cash inicial;
- SPY desde inception;
- allocation request;
- reserva;
- primera composición;
- primer snapshot.

No hardcodear:

capital = 10.000

Usar el capital elegido y confirmado por Omar.

==================================================
19. INICIO DEL TRACK DE CARTERA NUEVA
==================================================

La inception session de una cartera nueva será la sesión efectiva de su
aportación inicial.

Orden económico:

1. crear configuración;
2. registrar aportación inicial;
3. fijar Paper Unit Value = 100;
4. emitir unidades;
5. crear Paper Cash;
6. procesar primera compra confirmada;
7. crear primera composición EOD;
8. crear primer snapshot;
9. iniciar SPY con el mismo flujo externo;
10. hacer visible la cartera en LensPaperTrackView.

La aportación inicial y la compra:

- no crean retorno;
- no cambian el Unit Value inicial de 100.

Si la primera solicitud excede el capital:

- crear la cartera con su capital confirmado;
- dejar la solicitud PENDING_CAPITAL;
- iniciar el Track como cartera en cash únicamente si Omar confirma
  explícitamente la creación en ese estado.

No crear silenciosamente una cartera vacía.

==================================================
20. CARTERAS DINÁMICAS EN LA UI
==================================================

LensPaperTrackView no debe contener una lista hardcodeada de cinco carteras.

Crear o reutilizar un endpoint read-only que liste las carteras Papel:

GET /lens/paper-portfolios

Debe devolver:

- portfolio_id;
- slug;
- nombre;
- status;
- inception session;
- last certified session;
- Paper NAV;
- Paper Cash;
- posiciones activas;
- solicitudes pendientes;
- policy;
- coverage.

La vista debe:

- incluir automáticamente nuevas carteras activas;
- no requerir modificar HTML para cada nueva cartera;
- distinguir carteras activas, pendientes y archivadas;
- no mostrar una cartera fallida como plenamente operativa.

Los GET siguen siendo estrictamente read-only.

==================================================
21. ACTIVACIÓN DE LA POLÍTICA
==================================================

Activar:

MANUAL_NOTIONAL_HARD_CAP

solo después de que estén verdes:

- capacidad;
- reservas;
- concurrencia;
- preview;
- confirmación;
- pendientes;
- asignación parcial;
- aportación explícita;
- cartera nueva;
- ejecución EOD;
- actualización del Track;
- idempotencia;
- UI;
- Chrome;
- aislamiento económico.

La activación debe crear una nueva configuración versionada para cada cartera
existente.

La configuración histórica legacy:

- permanece vigente hasta el instante anterior;
- no se modifica;
- no se elimina;
- sigue disponible para replay.

La configuración prospectiva:

allocation_policy_id =
MANUAL_NOTIONAL_HARD_CAP

effective_from =
primera sesión NYSE segura posterior al despliegue.

Regla:

- si el despliegue y todas las verificaciones terminan antes de la apertura
  regular de una sesión, esa sesión puede ser effective_from;
- si terminan durante o después de la sesión, effective_from será la
  siguiente sesión NYSE;
- respetar early close, festivos, fines de semana y America/New_York.

No codificar una fecha fija.

Persistir:

- activated_at_utc;
- effective_from_session;
- approved_by = Omar;
- activation_reference;
- policy version;
- content hash.

No aplicar la política retroactivamente.

==================================================
22. TRANSICIÓN DE LAS CINCO CARTERAS EXISTENTES
==================================================

Al activar hard cap:

- conservar todas las posiciones existentes;
- conservar cantidades;
- conservar Paper NAV;
- conservar Paper Cash;
- conservar Paper Units;
- conservar Paper Unit Value;
- no crear transacciones;
- no crear retorno;
- no crear flujos;
- no rebalancear.

La nueva política regula únicamente:

- nuevas incorporaciones;
- ampliaciones;
- salidas;
- reasignaciones;
- aportaciones;
- futuras composiciones confirmadas.

Debe cumplirse:

NAV_before_policy_activation =
NAV_after_policy_activation

units_before =
units_after

unit_value_before =
unit_value_after

transactions_created_by_activation =
0

==================================================
23. SCHEMA
==================================================

Auditar si v29 soporta:

- cash reservation;
- confirmed_at;
- confirmed_by;
- effective session;
- partial confirmation;
- unallocated_notional;
- pending price;
- cancellation;
- failure reason;
- prospective composition;
- policy activation;
- dynamic portfolio creation;
- idempotency;
- reassignment group;
- external contributions;
- transaction supersession.

Si v29 ya lo soporta:

- no migrar.

Si falta soporte:

- crear una migración aditiva mínima;
- utilizar NEXT_AVAILABLE_SCHEMA_VERSION;
- no asumir un número;
- no modificar tablas reales;
- no reutilizar campos con semántica diferente;
- probar migración, rollback vacío, idempotencia y foreign keys.

Puede añadirse una tabla específica de reservas si es necesaria:

paper_allocation_reservation

Campos mínimos:

- reservation_id;
- allocation_request_id;
- paper_portfolio_id;
- amount;
- currency;
- status;
- reserved_at;
- expires_at nullable;
- released_at nullable;
- consumed_at nullable;
- idempotency_key;
- content_hash.

Estados:

- ACTIVE;
- CONSUMED;
- RELEASED;
- EXPIRED;
- INVALIDATED.

Una reserva activa debe ser única por solicitud.

==================================================
24. API
==================================================

Crear endpoints claros.

A. CARTERAS

GET /lens/paper-portfolios

POST /lens/paper-portfolios/preview

POST /lens/paper-portfolios

B. SOLICITUDES

POST /lens/paper-allocation-requests/preview

POST /lens/paper-allocation-requests

POST /lens/paper-allocation-requests/:id/confirm

POST /lens/paper-allocation-requests/:id/confirm-partial

POST /lens/paper-allocation-requests/:id/cancel

GET /lens/paper-allocation-requests

GET /lens/paper-allocation-requests/:id

C. FLUJOS

POST /lens/paper-flows/preview

POST /lens/paper-flows

D. SALIDAS

POST /lens/paper-exits/preview

POST /lens/paper-exits

E. REASIGNACIONES

POST /lens/paper-reassignments/preview

POST /lens/paper-reassignments

Mantener separados:

- preview;
- creación;
- confirmación;
- ejecución al cierre.

Los endpoints de escritura deben exigir:

- autenticación/autorización existente;
- CSRF o protección equivalente según la arquitectura;
- idempotency key;
- validación de payload;
- audit trail;
- respuesta determinista.

Los GET no pueden escribir.

==================================================
25. UI DE ASIGNACIÓN
==================================================

Desde una tesis aprobada, mostrar:

AÑADIR A CARTERA PAPEL

Paso 1:

- seleccionar cartera existente;
- o crear cartera nueva.

Paso 2:

- introducir requested_notional.

Paso 3:

- mostrar preview.

Paso 4:

- confirmar.

Preview para cartera existente:

- ticker;
- tesis;
- cartera;
- Paper NAV;
- Paper Cash bruto;
- cash reservado;
- capacidad disponible;
- requested_notional;
- allocated_notional propuesto;
- estado esperado;
- peso inicial estimado;
- Paper Cash posterior;
- sesión efectiva estimada;
- política;
- warnings.

Preview para cartera nueva:

- nombre;
- capital;
- requested_notional;
- capacidad;
- Paper Cash posterior;
- primera sesión;
- unidades iniciales;
- SPY;
- estado esperado.

No ejecutar desde el preview.

==================================================
26. COLA DE SOLICITUDES
==================================================

Crear una vista de solicitudes Papel.

Mostrar:

- ticker;
- tesis;
- cartera;
- requested_notional;
- allocated_notional;
- unallocated_notional;
- status;
- capacidad;
- sesión efectiva;
- fecha de confirmación;
- motivo de pending/failure;
- acciones disponibles.

Filtros:

- pendientes;
- reservadas;
- esperando cierre;
- esperando precio;
- completas;
- parciales;
- canceladas;
- fallidas.

Acciones contextuales:

- confirmar;
- confirmar parcial;
- aportar capital;
- cancelar;
- reintentar;
- abrir cartera;
- abrir tesis.

No permitir acciones incompatibles con el estado.

==================================================
27. INTEGRACIÓN CON DAILY-CLOSE
==================================================

Extender el adaptador Papel existente.

Orden por cartera y sesión:

1. extender y validar manifiesto;
2. aplicar flujos externos confirmados;
3. emitir/retirar unidades;
4. cargar solicitudes RESERVED_FOR_EXECUTION/PENDING_MARKET_CLOSE;
5. validar precio;
6. consumir reservas;
7. crear posiciones/memberships;
8. crear composición EOD;
9. crear transacciones internas;
10. comprobar NAV continuity;
11. crear snapshot;
12. actualizar estados de solicitudes;
13. exponer endpoint y gráfica.

Una única transacción debe agrupar los cambios económicos de una cartera y
sesión.

Si falla:

- rollback de esa cartera/sesión;
- no consumir reserva parcialmente;
- no crear media composición;
- no crear snapshot inconsistente;
- no impedir procesar otras carteras;
- registrar error;
- reintentar mediante catch-up.

==================================================
28. STARTUP CATCH-UP
==================================================

Al arrancar:

- detectar solicitudes confirmadas pendientes;
- detectar sesiones ya cerradas sin ejecución;
- procesarlas cronológicamente;
- extender manifiesto si procede;
- no duplicar;
- no utilizar precio live;
- no ejecutar una solicitud cancelada;
- no saltar una sesión bloqueada;
- no crear exposición antes de confirmed_at.

El catch-up debe poder recuperar:

- cierre ocurrido con servidor apagado;
- precio publicado con retraso;
- reinicio durante procesamiento;
- transacción interrumpida;
- snapshot pendiente.

==================================================
29. ACTUALIZACIÓN AUTOMÁTICA DE LA GRÁFICA
==================================================

Después de una ejecución correcta al cierre:

- el nuevo snapshot queda disponible en el endpoint;
- LensPaperTrackView lo muestra al recargar o refrescar sus datos;
- la nueva posición aparece desde su primera sesión económica;
- el Track anterior permanece intacto;
- SPY no recibe ningún flujo por la compra interna;
- Paper Unit Value no salta por la compra.

Para una cartera nueva:

- aparece automáticamente en GET /lens/paper-portfolios;
- aparece en el selector;
- muestra su primer snapshot;
- Paper Unit Value comienza en 100;
- SPY comienza en la misma inception session.

No modificar manualmente la gráfica.

==================================================
30. IDEMPOTENCIA Y CONCURRENCIA
==================================================

Probar:

- doble clic en confirmar;
- dos requests con misma idempotency key;
- dos requests compitiendo por el mismo cash;
- reinicio durante confirmación;
- reinicio durante daily-close;
- ejecución repetida de catch-up;
- precio recibido dos veces;
- snapshot ya existente;
- cartera nueva creada dos veces;
- cancelación simultánea con ejecución.

Resultados:

- una sola solicitud;
- una sola reserva;
- una sola posición;
- un solo membership;
- una sola transacción;
- una sola composición;
- un solo snapshot;
- capacidad nunca negativa.

La operación que pierde la carrera debe:

- quedar PENDING_CAPITAL;
- o devolver conflicto determinista;
- nunca sobregirar cash.

==================================================
31. SEGURIDAD Y AUDITORÍA
==================================================

Persistir:

- requested_by;
- confirmed_by;
- created_at;
- confirmed_at;
- cancelled_at;
- executed_at;
- effective_session;
- request payload hash;
- preview hash;
- policy id/version;
- source thesis;
- source catalyst cuando exista;
- idempotency key;
- execution reference;
- failure reason.

No guardar:

- secretos;
- tokens;
- datos innecesarios;
- payloads completos con información sensible.

Cada escritura económica debe ser trazable desde:

tesis
→ request
→ confirmación
→ reserva
→ flujo, cuando exista
→ composición
→ transacción
→ snapshot

==================================================
32. PRUEBAS UNITARIAS Y ECONÓMICAS
==================================================

Crear:

verify-paper-hard-cap-policy.mjs
verify-paper-capacity-reservation.mjs
verify-paper-allocation-preview.mjs
verify-paper-allocation-confirmation.mjs
verify-paper-partial-allocation.mjs
verify-paper-pending-capital.mjs
verify-paper-external-contribution.mjs
verify-paper-prospective-composition.mjs
verify-paper-prospective-transaction.mjs
verify-paper-new-portfolio.mjs
verify-paper-position-exit.mjs
verify-paper-prospective-reassignment.mjs
verify-paper-allocation-idempotency.mjs
verify-paper-allocation-concurrency.mjs
verify-paper-track-auto-update.mjs

Casos obligatorios:

A. Cash suficiente.

- asignación completa;
- requested preservado;
- reserva;
- compra al cierre;
- Track actualizado.

B. Cash insuficiente.

- PENDING_CAPITAL;
- cero posición;
- cero Track impact.

C. Posición 11.

- pending;
- cero dilución;
- cero rebalanceo.

D. Asignación parcial.

- requiere confirmación;
- requested permanece;
- diferencia visible.

E. Aportación externa.

- emite unidades;
- Unit Value neutral;
- SPY replica el flujo.

F. Compra interna.

- NAV neutral;
- Units neutral;
- Unit Value neutral.

G. Cartera nueva.

- capital configurable;
- Unit Value 100;
- primera compra;
- SPY;
- primer snapshot;
- aparece en selector.

H. Cancelación.

- libera reserva;
- cero exposición.

I. Falta de precio.

- PENDING_PRICE;
- no fallback;
- no compra.

J. After-close.

- siguiente sesión.

K. Early close.

- horario correcto.

L. Dos solicitudes.

- no doble gasto.

M. Doble confirmación.

- idempotente.

N. Salida.

- proceeds a Paper Cash;
- sin retirada externa.

O. Reasignación.

- atómica;
- hard cap destino;
- sin duplicación.

P. Histórico.

- snapshots anteriores intactos;
- legacy intacto.

Q. Gráfica.

- nuevo snapshot visible;
- cartera nueva visible.

==================================================
33. PRUEBAS UI Y CHROME
==================================================

Validar en Chrome real:

A. TESIS → CARTERA EXISTENTE

- abrir tesis;
- añadir a cartera;
- seleccionar cartera;
- introducir importe;
- preview;
- confirmar;
- ver estado pendiente;
- simular cierre en LAB;
- comprobar Track.

B. TESIS → CARTERA NUEVA

- crear cartera;
- indicar capital;
- indicar importe;
- preview;
- confirmar;
- simular cierre;
- comprobar selector;
- comprobar primera gráfica.

C. SIN CAPACIDAD

- PENDING_CAPITAL;
- posiciones existentes intactas;
- gráfica intacta.

D. PARCIAL

- segunda confirmación;
- diferencia visible.

E. CANCELACIÓN

- reserva liberada.

F. RESPONSIVE

- desktop;
- móvil;
- modal;
- preview;
- cola;
- errores.

No aceptar:

- confirmación implícita;
- doble submit;
- cash negativo;
- selector hardcodeado;
- cartera ausente después de crearla;
- posición visible antes del cierre;
- Track actualizado antes de la ejecución;
- errores JavaScript.

==================================================
34. ACTIVACIÓN Y DESPLIEGUE
==================================================

FASE A · LAB

- schema/migración, si procede;
- motor;
- APIs;
- UI;
- concurrencia;
- daily-close;
- catch-up;
- Track;
- Chrome;
- regresión.

FASE B · PRODUCCIÓN SIN ACTIVAR

- desplegar código;
- política sigue pendiente;
- verificar health;
- verificar UI;
- verificar endpoints;
- verificar que ninguna tesis antigua se asigna.

FASE C · ACTIVACIÓN

Solo tras Fase B verde:

- calcular safe effective_from;
- crear configuraciones prospectivas versionadas;
- activar MANUAL_NOTIONAL_HARD_CAP;
- registrar aprobación;
- verificar cero transacciones creadas por activación.

FASE D · SMOKE TEST CONTROLADO

No utilizar una posición real ni generar impacto real.

Realizar un caso papel controlado:

- crear request de prueba o fixture autorizado;
- preview;
- confirmar/cancelar antes del cierre;
- comprobar reserva y liberación;

o ejecutar íntegramente en LAB si Producción no dispone de un ticker de prueba
autorizado.

No contaminar el Track productivo con datos ficticios.

==================================================
35. ROLLBACK
==================================================

Después de activar, no borrar historia.

Rollback operativo:

- impedir nuevas confirmaciones;
- marcar política como SUSPENDED o equivalente;
- mantener requests existentes visibles;
- no borrar composiciones;
- no borrar transacciones;
- no borrar snapshots;
- no volver automáticamente a equal-weight;
- aplicar migración correctiva hacia delante si fuera necesaria.

Las solicitudes confirmadas pendientes deben quedar:

- pausadas;
- cancelables;
- auditables.

Documentar:

- feature disable;
- policy suspension;
- recuperación;
- reconciliación;
- reactivación.

==================================================
36. REGRESIÓN E INTEGRIDAD
==================================================

Ejecutar:

- suites v27/v28/v29;
- migración nueva, si existe;
- Gate 2C-2;
- Gate 2D-1;
- Gate 2D-2;
- Gate 2E-A;
- Gate 2E-B;
- financial core;
- daily-close;
- startup catch-up;
- repository integrity;
- tesis;
- catalizadores;
- Campo de Caza;
- Track Papel.

Comparar antes/después:

- datos legacy;
- snapshots históricos Papel;
- reconstrucción económica;
- Book real;
- cash real;
- NAV real;
- valuations reales;
- movimientos;
- decisiones;
- tesis históricas;
- catalizadores.

Resultados obligatorios:

HISTORICAL_PAPER_TRACK_UNCHANGED = PASS
REAL_BOOK_UNCHANGED = PASS
REAL_CASH_UNCHANGED = PASS
REAL_NAV_UNCHANGED = PASS
REAL_VALUATIONS_UNCHANGED = PASS
LEGACY_HOLDINGS_UNCHANGED = PASS
LEGACY_SNAPSHOTS_UNCHANGED = PASS
HISTORICAL_THESES_UNCHANGED = PASS
CATALYSTS_UNCHANGED = PASS
HUNTING_FIELD_UNCHANGED = PASS

==================================================
37. CÓDIGO Y COMMIT
==================================================

Añadir:

- hard-cap engine;
- capacity/reservations;
- request lifecycle;
- previews;
- confirmations;
- pending queue;
- portfolio creation;
- external contributions;
- exits;
- prospective compositions;
- prospective transactions;
- daily-close integration;
- catch-up;
- dynamic portfolio listing;
- UI;
- tests;
- rollback documentation.

No añadir:

- scratchpad;
- backups;
- bases;
- secretos;
- datos productivos;
- fixtures productivos falsos;
- tags.

Ejecutar secret scan.

Crear un único commit local:

feat(lens): activate manual hard-cap paper allocation workflow

No hacer push.
No crear tag.

==================================================
38. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = 0466595
CANONICAL_HEAD_AFTER = <hash>

SCHEMA_BEFORE = v29
SCHEMA_AFTER = <v29/NEXT_AVAILABLE_SCHEMA_VERSION>

PROSPECTIVE_ALLOCATION_POLICY =
MANUAL_NOTIONAL_HARD_CAP

PROSPECTIVE_POLICY_STATUS =
ACTIVE /
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
<sesión/NULL>

PROSPECTIVE_AUTO_REBALANCE =
DISABLED

PARTIAL_ALLOCATION_POLICY =
REQUIRE_EXPLICIT_CONFIRMATION

LEVERAGE_POLICY =
DISABLED

EXISTING_PORTFOLIOS_CONFIGURED = <número>/5
DYNAMIC_PORTFOLIO_CREATION = PASS/FAIL
DYNAMIC_PORTFOLIO_LISTING = PASS/FAIL

THESIS_TO_ALLOCATION_REQUEST = PASS/FAIL
ALLOCATION_PREVIEW = PASS/FAIL
EXPLICIT_CONFIRMATION = PASS/FAIL
CAPACITY_RESERVATION = PASS/FAIL
CAPACITY_RACE_PROTECTION = PASS/FAIL

FULL_ALLOCATION = PASS/FAIL
PENDING_CAPITAL = PASS/FAIL
PARTIAL_ALLOCATION = PASS/FAIL
PARTIAL_CONFIRMATION_REQUIRED = PASS/FAIL
CANCELLATION_RELEASES_RESERVATION = PASS/FAIL

POSITION_11_PENDING = PASS/FAIL
EXISTING_POSITIONS_NOT_REBALANCED = PASS/FAIL
NO_AUTOMATIC_NORMALIZATION = PASS/FAIL
NO_IMPLICIT_CONTRIBUTION = PASS/FAIL
NO_NEGATIVE_CASH = PASS/FAIL
NO_LEVERAGE = PASS/FAIL

EXTERNAL_CONTRIBUTION_UNIT_NEUTRAL = PASS/FAIL
INTERNAL_PURCHASE_NAV_NEUTRAL = PASS/FAIL
INTERNAL_PURCHASE_UNIT_NEUTRAL = PASS/FAIL
POSITION_EXIT_KEEPS_PAPER_CASH = PASS/FAIL
PROSPECTIVE_REASSIGNMENT_ATOMIC = PASS/FAIL

NEW_PORTFOLIO_INITIAL_UNIT_VALUE = 100
NEW_PORTFOLIO_SPY_FROM_INCEPTION = PASS/FAIL
NEW_PORTFOLIO_FIRST_SNAPSHOT = PASS/FAIL
NEW_PORTFOLIO_VISIBLE_IN_SELECTOR = PASS/FAIL

DAILY_CLOSE_EXECUTES_CONFIRMED_REQUESTS = PASS/FAIL
STARTUP_CATCHUP_EXECUTES_MISSED_REQUESTS = PASS/FAIL
PENDING_PRICE_FAIL_CLOSED = PASS/FAIL

TRACK_AUTO_UPDATE_EXISTING_PORTFOLIO = PASS/FAIL
TRACK_AUTO_UPDATE_NEW_PORTFOLIO = PASS/FAIL
HISTORICAL_PAPER_TRACK_UNCHANGED = PASS/FAIL

REQUEST_IDEMPOTENCY = PASS/FAIL
RESERVATION_IDEMPOTENCY = PASS/FAIL
EXECUTION_IDEMPOTENCY = PASS/FAIL
CONCURRENCY_SAFE = PASS/FAIL

UI_THESIS_EXISTING_PORTFOLIO = PASS/FAIL
UI_THESIS_NEW_PORTFOLIO = PASS/FAIL
UI_PENDING_QUEUE = PASS/FAIL
UI_PARTIAL_CONFIRMATION = PASS/FAIL
UI_CANCELLATION = PASS/FAIL
CHROME_GREEN = PASS/FAIL
JAVASCRIPT_ERRORS = <número>

PAPER_TRACK_VIEW_UNCHANGED_EXCEPT_DYNAMIC_PORTFOLIOS = PASS/FAIL

REAL_BOOK_UNCHANGED = PASS/FAIL
REAL_CASH_UNCHANGED = PASS/FAIL
REAL_NAV_UNCHANGED = PASS/FAIL
REAL_VALUATIONS_UNCHANGED = PASS/FAIL
LEGACY_HOLDINGS_UNCHANGED = PASS/FAIL
LEGACY_SNAPSHOTS_UNCHANGED = PASS/FAIL
CATALYSTS_UNCHANGED = PASS/FAIL
HUNTING_FIELD_UNCHANGED = PASS/FAIL
NO_REAL_ECONOMIC_WRITES = PASS/FAIL

POLICY_ACTIVATION_NAV_CONTINUITY = PASS/FAIL
POLICY_ACTIVATION_UNIT_CONTINUITY = PASS/FAIL
POLICY_ACTIVATION_TRANSACTIONS_CREATED = 0

ROLLBACK_DOCUMENTED = PASS/FAIL
SECRET_SCAN = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

LOCAL_COMMIT = <hash>

EXACT_REMAINING_BLOCKERS:
1. <bloqueo>
2. <bloqueo>
3. <bloqueo>

Detenerse después de Gate 2F.