MIZAN · LA LENTE
GATE 2A · CIERRE METODOLÓGICO Y RECONCILIACIÓN DE EPISODIOS
PRECONDICIÓN DEL GATE 2B

NATURALEZA

Esta ejecución es exclusivamente:

- corrección del inventario histórico;
- reconciliación de episodios por cartera;
- cierre de decisiones metodológicas;
- propuesta final de v27;
- simulación LAB.

No autoriza:

- migración;
- cambio de schema productivo;
- backfill;
- fetch masivo de precios;
- modificación de holdings;
- cambio de UI;
- deploy;
- commit;
- tag;
- push.

==================================================
0. DECISIONES DE OMAR
==================================================

Quedan aprobadas estas decisiones:

PROSPECTIVE_ALLOCATION_POLICY =
TARGET_NOTIONAL_PRO_RATA

Condiciones:

- requested_notional se conserva inmutable;
- allocated_notional se versiona;
- se conserva la proporción entre solicitudes;
- todo cambio de asignación requiere preview;
- todo rebalanceo requiere confirmación expresa;
- no existe aportación implícita;
- no existe leverage implícito;
- no existe cash negativo;
- no se reescribe la composición histórica.

MANUAL_NOTIONAL_HARD_CAP puede existir como configuración alternativa por
cartera, pero no será la política predeterminada de las carteras catalizadas.

La reconstrucción histórica aprobada es:

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE_AS_OPERATED

La historia debe representar cada composición y rebalanceo en el momento en
que ocurrió.

No aplicar TARGET_NOTIONAL_PRO_RATA retroactivamente.

Metodología aprobada:

RETURN_METHOD =
PRICE_RETURN_TWR

EXIT_POLICY =
KEEP_AS_PAPER_CASH

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

COMMISSION_POLICY =
NOT_MODELED

BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

==================================================
1. CORREGIR LA IDENTIDAD DE EPISODIOS
==================================================

No confundir:

- holdings globales abiertos;
- solicitudes de asignación;
- episodios de pertenencia a una cartera;
- composición vigente;
- posición económica subyacente.

Se han identificado:

- 82 holdings abiertos actuales;
- 39 reasignaciones desde cat:catalizada a otras carteras;
- 0 ventas económicas globales.

Determinar separadamente:

A. GLOBAL_PAPER_POSITION

Representa la continuidad de la posición o tesis papel subyacente.

B. PORTFOLIO_MEMBERSHIP_EPISODE

Representa el intervalo durante el que una posición perteneció a una cartera
concreta.

Una reasignación debe:

1. mantener la posición global abierta;
2. cerrar el membership episode en la cartera de origen;
3. abrir un membership episode en la cartera de destino;
4. conservar timestamp y evento de reasignación;
5. no reescribir el periodo anterior;
6. no considerarse aportación o retirada externa.

Recalcular:

GLOBAL_PAPER_POSITIONS
CURRENT_OPEN_GLOBAL_POSITIONS
PORTFOLIO_MEMBERSHIP_EPISODES
CURRENT_OPEN_MEMBERSHIP_EPISODES
CLOSED_MEMBERSHIP_EPISODES
ECONOMICALLY_CLOSED_GLOBAL_POSITIONS
INTER_PORTFOLIO_REASSIGNMENTS
REINCORPORATIONS
UNRESOLVED_EPISODE_LINEAGE

No declarar PAPER_EFFECTIVE_EPISODES = 82 sin aclarar si el dato se refiere a
posiciones globales o memberships por cartera.

==================================================
2. SEMÁNTICA DE REASIGNACIÓN
==================================================

Para cada una de las 39 reasignaciones reconstruir:

- global_position_id;
- source_holding_id;
- ticker;
- thesis_id;
- origin_portfolio_id;
- destination_portfolio_id;
- reassigned_at_utc;
- New York local datetime;
- effective_session;
- origin composition before;
- origin composition after;
- destination composition before;
- destination composition after;
- origin allocated notional before;
- origin allocated notional after;
- destination allocated notional before;
- destination allocated notional after;
- origin transactions required;
- destination transactions required;
- requested_notional;
- requested_notional_status;
- source log event ids.

Regla económica:

En la cartera de origen:

- la posición deja de formar parte de la composición;
- se registra el rebalanceo interno correspondiente;
- el capital permanece dentro de esa cartera como paper cash o se reasigna
  según la composición legacy vigente;
- no se retiran unidades;
- no existe flujo externo.

En la cartera de destino:

- la posición entra en la composición;
- se registra el rebalanceo interno legacy correspondiente;
- se financia con paper cash o con ventas internas de la composición;
- no se emiten unidades;
- no existe flujo externo.

La reasignación no transfiere unidades TWR entre carteras.

Cada cartera conserva:

- su propio capital configurado;
- sus propias unidades;
- su propio NAV;
- su propio benchmark.

No modelar la reasignación como una venta económica global de la tesis.

==================================================
3. EVENTO Y PRECIO EFECTIVO DE REASIGNACIÓN
==================================================

Resolver el instante efectivo mediante:

cat_composicion_log timestamp
→ UTC
→ America/New_York
→ market state
→ effective_session

Si la reasignación ocurrió durante mercado abierto:

- las composiciones nuevas son efectivas al cierre;
- usar el cierre oficial de esa sesión;
- no aplicar la nueva composición antes del cierre.

Si ocurrió después del cierre:

- usar la última sesión completamente cerrada;
- hacer efectiva la composición desde el instante de reasignación.

No inventar precios.

Si falta cobertura:

REASSIGNMENT_PRICE_COVERAGE_REQUIRED

Estos precios se obtendrán posteriormente en Gate 2C.

==================================================
4. LOS 36 REQUESTED_NOTIONAL DESCONOCIDOS
==================================================

El inventario correcto es:

- 46 solicitudes con requested_notional recuperable del log;
- 36 sin requested_notional recuperable;
- dentro de esos 36:
  - 26 incorporaciones anteriores al inicio del log;
  - 10 sin vínculo suficiente con thesis_added.

Para los 36 casos:

requested_notional = NULL

requested_notional_status =
UNKNOWN_PRE_LOG
o
UNKNOWN_UNLINKED

No utilizar como requested_notional:

- allocated_notional actual;
- capital/N;
- original_entry_price × quantity;
- 1.000 por analogía;
- el importe de otra posición.

Puede reconstruirse allocated_notional histórico cuando sea determinista por:

configured_capital / active_membership_count

Persistir en la propuesta futura:

- requested_notional;
- requested_notional_status;
- allocated_notional;
- allocation_source;
- reconstruction_confidence.

Valores de allocation_source:

- LOGGED_REQUEST;
- LEGACY_EQUAL_WEIGHT_RECONSTRUCTION;
- CURRENT_HOLDING_STATE;
- UNKNOWN.

Valores de reconstruction_confidence:

- PROVEN;
- DETERMINISTIC;
- INFERRED;
- UNKNOWN.

No bloquear el Track histórico únicamente porque requested_notional sea
desconocido si allocated_notional puede reconstruirse determinísticamente.

==================================================
5. REPLAY HISTÓRICO AS-OPERATED
==================================================

Reconstruir por cartera todas las versiones de composición.

Orden:

1. timestamp absoluto;
2. sequence/id del log;
3. regla de desempate documentada.

Para cada versión registrar:

- portfolio_id;
- composition_version;
- effective_at;
- effective_session;
- configured_capital;
- active memberships;
- legacy N;
- requested_notional conocido o null;
- allocated_notional antes;
- allocated_notional después;
- quantity antes;
- quantity propuesta después;
- rebalance transactions;
- paper cash;
- source event;
- confidence.

Durante la política legacy:

legacy_allocated_notional =
configured_capital / active_membership_count

salvo que exista evidencia de pesos explícitos diferentes.

No aplicar la composición final hacia atrás.

Una incorporación, retirada de membership o reasignación puede provocar un
rebalanceo secuencial.

La reconstrucción debe reflejarlo en la sesión efectiva correspondiente.

==================================================
6. TRATAMIENTO DEL PERIODO PRE-LOG
==================================================

Para las incorporaciones anteriores a cat_composicion_log:

Usar como fuentes:

- holdings.entrada_ts;
- cartera de origen demostrable;
- configured capital;
- cantidad de memberships activas según entrada_ts;
- código legacy aplicable en esa fecha;
- snapshots existentes;
- precios originales conservados.

Antes de asumir que hubo rebalanceo capital/N en cada alta, demostrar:

- que reescalarCarteraPapel ya existía;
- que era llamado por ese mismo code path;
- que la política no cambió entre las fechas pre-log y el inicio del log.

Clasificar cada versión pre-log:

PRE_LOG_POLICY_PROVEN
PRE_LOG_POLICY_DETERMINISTIC
PRE_LOG_POLICY_INFERRED
PRE_LOG_POLICY_UNKNOWN

Solo las dos primeras pueden ser candidatas a backfill automático.

Las inferidas o desconocidas requieren warning y revisión.

No inventar requested_notional.

==================================================
7. NOTIONAL Y CANTIDAD CERTIFICADOS
==================================================

Para la historia legacy:

certified_allocated_notional =
allocated_notional vigente en cada composition version.

En la incorporación inicial o en un rebalanceo:

certified_quantity_after =
certified_allocated_notional_after
/
official_close de la effective_session

Conservar:

- original_quantity;
- original_entry_price;
- original_notional;
- requested_notional;
- requested_notional_status;
- allocated_notional_before;
- allocated_notional_after;
- certified_quantity_before;
- certified_quantity_after.

No sobrescribir datos originales.

No asumir que requested_notional es el capital económico efectivamente
asignado.

Para las 46 solicitudes conocidas:

- requested_notional conserva 1.000;
- allocated_notional refleja la normalización legacy vigente.

Para las 36 desconocidas:

- requested_notional permanece null;
- allocated_notional se reconstruye únicamente cuando sea determinista.

==================================================
8. POLÍTICA PROSPECTIVA SELLABLE
==================================================

Desde una fecha futura de activación:

TARGET_NOTIONAL_PRO_RATA_CONFIRMED

Fórmula:

total_requested =
suma de requested_notional activos y válidos

Si total_requested <= configured_capital:

scale_factor = 1

allocated_notional_i =
requested_notional_i

paper_cash =
configured_capital - total_requested

Si total_requested > configured_capital:

scale_factor =
configured_capital / total_requested

allocated_notional_i =
requested_notional_i × scale_factor

paper_cash =
0, salvo diferencias de redondeo.

La política debe:

- conservar requested_notional;
- conservar proporciones;
- mostrar allocated_notional antes/después;
- mostrar cantidades antes/después;
- mostrar transacciones;
- mostrar turnover;
- mostrar paper cash;
- exigir confirmación;
- crear una nueva composition version;
- no modificar versiones anteriores;
- no crear flujos externos;
- no cambiar unidades TWR.

El cambio de legacy a pro-rata debe registrarse mediante:

- policy effective_from;
- preview;
- confirmación;
- composición versionada;
- rebalanceo interno si procede.

==================================================
9. REVISIÓN DEL SCHEMA v27
==================================================

La propuesta final debe distinguir:

A. paper_global_position

Continuidad de la posición o tesis papel subyacente.

B. paper_portfolio_membership_episode

Intervalo de pertenencia de la posición a una cartera.

Campos mínimos:

- membership_episode_id;
- global_position_id;
- paper_portfolio_id;
- source_holding_id;
- entered_at_utc;
- entered_session;
- exited_at_utc;
- exited_session;
- entry_reason;
- exit_reason;
- origin_membership_episode_id;
- destination_membership_episode_id;
- requested_notional;
- requested_notional_status;
- status;
- content_hash.

C. paper_track_methodology

Metodología de retorno y precios, sin capital universal.

D. paper_portfolio_track_config

Capital y política por cartera, versionados.

E. paper_allocation_request

Solicitud original, nullable cuando no sea recuperable.

F. paper_composition_version

Composición vigente por intervalo.

G. paper_composition_member

Asignación de cada membership en una versión.

H. paper_portfolio_flow

Solo flujos externos.

I. paper_portfolio_transaction

Compras, ventas y rebalanceos internos.

J. paper_track_snapshot

Serie unitizada y benchmark.

Simplificar únicamente si puede conservarse:

- continuidad global;
- pertenencia por cartera;
- composición point-in-time;
- solicitudes;
- asignaciones;
- transacciones;
- trazabilidad.

No aplicar la migración.

==================================================
10. PRUEBAS LAB
==================================================

A. REASIGNACIÓN ENTRE CARTERAS

- global position continúa abierta;
- membership origen se cierra;
- membership destino se abre;
- cero flujo externo;
- cero transferencia de unidades;
- ambas composiciones se versionan.

B. CONTEO DE EPISODIOS

- 82 posiciones globales abiertas;
- 39 reasignaciones;
- número de membership episodes consistente;
- memberships abiertos = holdings actuales;
- memberships cerrados = salidas históricas de composición.

C. REQUESTED DESCONOCIDO

- requested_notional = null;
- allocated_notional determinista permitido;
- no inventar 1.000.

D. PRE-LOG

- distinguir proven/deterministic/inferred/unknown;
- no autobackfill de casos inferidos.

E. REBALANCEO LEGACY

- composición anterior permanece intacta;
- nueva composición capital/N desde effective_session;
- transacciones internas;
- cero flujo externo;
- unidades sin cambio.

F. CAMBIO PROSPECTIVO

- pro-rata conserva proporciones;
- requiere confirmación;
- no reescribe legacy;
- unit value continuo.

FAIL = 0
NO_RESULT = 0

==================================================
11. ENTREGA
==================================================

GLOBAL_PAPER_POSITIONS = <número>
CURRENT_OPEN_GLOBAL_POSITIONS = <número>

PORTFOLIO_MEMBERSHIP_EPISODES = <número>
CURRENT_OPEN_MEMBERSHIP_EPISODES = <número>
CLOSED_MEMBERSHIP_EPISODES = <número>
ECONOMICALLY_CLOSED_GLOBAL_POSITIONS = <número>
INTER_PORTFOLIO_REASSIGNMENTS = <número>
UNRESOLVED_EPISODE_LINEAGE = <número>

REQUESTED_NOTIONAL_KNOWN = 46
REQUESTED_NOTIONAL_UNKNOWN = 36
REQUESTED_UNKNOWN_PRE_LOG = 26
REQUESTED_UNKNOWN_UNLINKED = 10

PRE_LOG_POLICY_PROVEN = <número>
PRE_LOG_POLICY_DETERMINISTIC = <número>
PRE_LOG_POLICY_INFERRED = <número>
PRE_LOG_POLICY_UNKNOWN = <número>

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE_AS_OPERATED

PROSPECTIVE_ALLOCATION_POLICY =
TARGET_NOTIONAL_PRO_RATA_CONFIRMED

RETURN_METHOD =
PRICE_RETURN_TWR

EXIT_POLICY =
KEEP_AS_PAPER_CASH

DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

COMMISSION_POLICY =
NOT_MODELED

BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

V27_GLOBAL_POSITION_MODELED = PASS/FAIL
V27_MEMBERSHIP_EPISODES_MODELED = PASS/FAIL
V27_COMPOSITION_VERSIONING_MODELED = PASS/FAIL
V27_REQUESTED_UNKNOWN_SUPPORTED = PASS/FAIL
V27_PRE_LOG_CONFIDENCE_SUPPORTED = PASS/FAIL
V27_FINAL_PROPOSAL = <ruta>

NO_PRODUCTION_SCHEMA_CHANGE = PASS/FAIL
NO_BACKFILL_EXECUTED = PASS/FAIL
NO_PAPER_HISTORY_MUTATED = PASS/FAIL
NO_REAL_DATA_MUTATED = PASS/FAIL
NO_ECONOMIC_WRITES = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

EXACT_BLOCKERS_BEFORE_GATE_2B:
1. <bloqueo real pendiente>
2. <bloqueo real pendiente>
3. <bloqueo real pendiente>

Detenerse.