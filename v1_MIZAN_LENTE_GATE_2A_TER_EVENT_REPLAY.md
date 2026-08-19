MIZAN · LA LENTE
GATE 2A-TER · REPLAY HISTÓRICO DE COMPOSICIÓN
POLÍTICA PROPORCIONAL PROSPECTIVA + IDENTIDAD DEFINITIVA DE EPISODIOS

NATURALEZA

Auditoría, replay point-in-time y simulación LAB.

No autoriza:

- migración v27;
- modificación de Producción;
- backfill;
- fetch masivo de precios;
- metodología SEALED;
- cambio de UI;
- cambio de daily-close;
- deploy;
- commit;
- tag;
- push.

Detenerse después de entregar el replay y la propuesta definitiva.

==================================================
0. HALLAZGOS CONFIRMADOS
==================================================

- carteras_papel.importe contiene el capital configurado;
- POST /tesis permite a Omar introducir un notional manual;
- cat_composicion_log conserva al menos parte del requested_notional;
- reescalarCarteraPapel sobrescribió holdings.importe y cantidad;
- la normalización actual utiliza capital/N;
- las asignaciones históricas actuales no conservan la solicitud manual;
- la normalización fue in-place y no point-in-time;
- las cinco carteras actuales suman aproximadamente su capital configurado;
- no hubo aportaciones implícitas ni leverage;
- los 82 holdings actuales no demuestran que solo existieran 82 episodios
  históricos;
- cat:catalizada contiene 46 eventos thesis_added, pero solo 17 holdings
  actuales;
- la declaración “0 salidas históricas” debe volver a verificarse contra todo
  cat_composicion_log.

==================================================
1. DECISIONES METODOLÓGICAS PREAPROBADAS
==================================================

POLÍTICA PROSPECTIVA RECOMENDADA:

allocation_policy =
TARGET_NOTIONAL_PRO_RATA

Semántica:

1. Omar introduce requested_notional por stock.
2. Mizan conserva ese valor de forma inmutable.
3. Si la suma de solicitudes activas no supera el capital:
   allocated_notional = requested_notional.
4. Si la suma supera el capital:
   scale_factor =
   configured_capital / total_active_requested_notional

   allocated_notional_i =
   requested_notional_i × scale_factor
5. La proporción entre solicitudes debe conservarse.
6. La incorporación que cambia el scale_factor requiere:
   - preview;
   - confirmación explícita de Omar;
   - rebalanceo point-in-time;
   - transacciones internas fechadas;
   - conservación de cantidades y asignaciones anteriores.
7. No crear aportaciones implícitas.
8. No permitir cash negativo.
9. No permitir leverage implícito.
10. No reescribir la historia anterior al rebalanceo.

Ejemplo:

capital = 10.000
25 solicitudes de 1.000
total solicitado = 25.000
scale_factor = 0,40
allocated_notional = 400 por stock

No dejar 15 stocks pendientes si Omar confirma que quiere que los 25 formen
la cartera.

POLÍTICA ALTERNATIVA CONFIGURABLE:

MANUAL_NOTIONAL_HARD_CAP

Puede existir como opción de cartera:

- respeta solicitudes hasta agotar cash;
- las siguientes quedan PENDING_CAPITAL;
- no es la política predeterminada para las carteras catalizadas actuales.

No utilizar el equal-weight actual capital/N cuando los requested_notional
sean distintos.

==================================================
2. SEPARACIÓN CANÓNICA DE DATOS
==================================================

Distinguir permanentemente:

requested_notional

- importe solicitado por Omar;
- nunca se sobrescribe por una normalización.

allocated_notional

- importe realmente asignado en una versión de composición;
- puede cambiar mediante un rebalanceo explícito.

original_quantity

- cantidad registrada históricamente.

certified_quantity

- cantidad correspondiente a allocated_notional y cierre oficial en el
  evento point-in-time correspondiente.

configured_capital

- capital externo configurado para la cartera.

paper_cash

- capital no asignado.

allocation_scale_factor

- factor aplicado a las solicitudes activas.

composition_version

- versión de composición efectiva desde una fecha y hora.

No utilizar holdings.importe como única fuente para requested_notional y
allocated_notional.

==================================================
3. REPLAY COMPLETO DE CAT_COMPOSICION_LOG
==================================================

Inventariar los 378 registros de cat_composicion_log.

Entregar por tipo de evento:

- event_type;
- número;
- primera fecha;
- última fecha;
- carteras afectadas;
- campos disponibles;
- capacidad para reconstrucción.

Identificar todos los tipos, incluyendo cuando existan:

- thesis_added;
- assigned;
- unassigned;
- removed;
- discarded;
- restored;
- portfolio_created;
- capital_changed;
- rebalanced;
- snapshot_created;
- thesis_closed;
- cualquier otro.

No limitar el replay a thesis_added.

Ordenar por:

1. timestamp absoluto;
2. sequence/id como desempate determinista.

Para cada evento determinar:

- cartera;
- ticker;
- thesis_id;
- requested_notional;
- allocated_notional antes;
- allocated_notional después;
- capital configurado;
- número de solicitudes activas;
- scale_factor;
- composición antes;
- composición después;
- cantidades antes;
- cantidades después;
- motivo;
- función de código que lo produjo.

==================================================
4. EXPLICAR 46 EVENTOS FRENTE A 17 HOLDINGS
==================================================

Para cat:catalizada reconciliar:

- 46 thesis_added;
- 17 holdings actuales;
- snapshots;
- asignaciones;
- descartes;
- tesis abiertas;
- cambios de cartera;
- cualquier retirada lógica.

Cada uno de los 46 eventos debe terminar clasificado como:

A. ACTIVE_CURRENT_HOLDING
B. REMOVED_FROM_PAPER_PORTFOLIO
C. REASSIGNED_TO_ANOTHER_PORTFOLIO
D. DISCARDED_BEFORE_EFFECTIVE_ALLOCATION
E. DUPLICATE_OR_IDEMPOTENT_EVENT
F. SUPERSEDED_COMPOSITION_REQUEST
G. UNRESOLVED

No declarar “0 salidas” solo porque no exista precio_salida.

Distinguir:

PAPER_POSITION_SALE

de:

PAPER_ALLOCATION_REMOVAL

Una eliminación de la composición puede cerrar un episodio papel aunque no
exista una operación real ni un campo clásico precio_salida.

Si no existe precio de salida:

- no inventarlo;
- marcar EXIT_PRICE_COVERAGE_REQUIRED;
- resolver posteriormente mediante cierre oficial de la sesión efectiva.

==================================================
5. IDENTIDAD DEFINITIVA DE EPISODIOS
==================================================

Recalcular:

PAPER_HISTORICAL_ALLOCATION_REQUESTS
PAPER_EFFECTIVE_EPISODES
PAPER_CURRENT_OPEN_EPISODES
PAPER_CLOSED_OR_REMOVED_EPISODES
PAPER_NEVER_EFFECTIVE_REQUESTS
PAPER_REINCORPORATIONS
PAPER_UNRESOLVED_LINEAGE

No utilizar 82 como número definitivo hasta reconciliar el log completo.

Un episodio económico empieza cuando:

- la solicitud fue aceptada en una composición;
- allocated_notional > 0;
- existe effective_at determinista.

Una solicitud no crea episodio cuando:

- fue descartada antes de asignarse;
- quedó pendiente;
- nunca tuvo allocated_notional;
- fue duplicada idempotentemente.

Una reincorporación crea un episodio nuevo.

==================================================
6. RECONSTRUCCIÓN DE LA POLÍTICA HISTÓRICA REAL
==================================================

El Track certificado histórico debe representar AS_OPERATED:

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

Para cada evento histórico que ejecutó reescalarCarteraPapel:

1. reconstruir composición inmediatamente anterior;
2. incorporar o retirar el ticker correspondiente;
3. calcular N activo;
4. calcular legacy_target_notional = configured_capital / N;
5. reconstruir allocated_notional antes y después para cada ticker;
6. registrar el rebalanceo point-in-time;
7. conservar requested_notional del log;
8. no aplicar retroactivamente la composición final.

No crear un único rebalanceo final si el código reescaló tras cada evento.

Ejemplo:

- 10 posiciones → 1.000 cada una;
- entra la posición 11;
- desde ese instante → aproximadamente 909,09 cada una;
- entra la posición 12;
- desde ese instante → aproximadamente 833,33 cada una;
- etc.

La historia anterior a cada incorporación conserva la asignación vigente en
ese momento.

==================================================
7. POLÍTICA PROSPECTIVA
==================================================

Aplicar únicamente desde una fecha de activación futura:

PROSPECTIVE_ALLOCATION_POLICY =
TARGET_NOTIONAL_PRO_RATA

No utilizarla para reescribir periodos históricos en los que Mizan aplicó
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE.

La metodología/configuración debe registrar:

- policy_id;
- version;
- effective_from;
- supersedes_policy_id;
- configured_capital;
- requested_notional semantics;
- allocation formula;
- confirmation requirement;
- leverage policy;
- partial allocation policy.

Cuando entra un stock nuevo y cambia la composición:

1. calcular preview;
2. mostrar requested_notional de todos;
3. mostrar allocated_notional antes/después;
4. mostrar compras/ventas necesarias;
5. mostrar pesos antes/después;
6. mostrar paper_cash;
7. solicitar confirmación;
8. solo tras confirmación crear versión de composición y transacciones;
9. no modificar episodios o cantidades anteriores in-place.

==================================================
8. COMPARACIÓN OBLIGATORIA DE POLÍTICAS
==================================================

Simular el ejemplo:

capital = 10.000
25 stocks
requested_notional = 1.000 cada uno

A. LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

- resultado final = 400 cada uno;
- rebalanceo automático en cada alta;
- requested_notional histórico preservado solo en el log.

B. TARGET_NOTIONAL_PRO_RATA

- resultado final = 400 cada uno;
- mismo resultado final en este caso;
- diferencia: conserva requested_notional y requiere confirmación;
- con solicitudes desiguales conserva proporciones.

C. MANUAL_NOTIONAL_HARD_CAP

- 10 activos;
- 15 pending;
- no representa el deseo de incluir los 25 salvo elección explícita.

Simular también:

capital = 10.000
solicitudes = 2.000, 500, 1.250, 8.000

total = 11.750
scale_factor = 10.000 / 11.750

TARGET_NOTIONAL_PRO_RATA debe conservar las proporciones.

LEGACY_EQUAL_WEIGHT no las conserva.

Entregar diferencias de:

- pesos;
- cash;
- turnover;
- transacciones;
- retorno;
- drawdown.

==================================================
9. TRACK HISTÓRICO Y TRACK PROSPECTIVO
==================================================

Definir una única serie continua con metodología versionada.

Antes de la activación:

allocation_methodology =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE_AS_OPERATED

Después de la activación:

allocation_methodology =
TARGET_NOTIONAL_PRO_RATA_CONFIRMED

No reiniciar unit value al cambiar de política.

El cambio de política:

- no es flujo externo;
- no emite ni retira unidades;
- puede provocar un rebalanceo interno;
- se registra en una sesión concreta;
- conserva continuidad del PRICE_RETURN_TWR.

Mantener:

EXIT_POLICY = KEEP_AS_PAPER_CASH
DIVIDEND_POLICY = INCOME_NOT_INCLUDED
COMMISSION_POLICY = NOT_MODELED
BENCHMARK_POLICY = EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

==================================================
10. PROPUESTA v27 REVISADA
==================================================

Diseñar, sin aplicar, soporte para:

A. paper_track_methodology

- metodología de retorno;
- anchors;
- precios;
- dividendos;
- comisiones;
- benchmark;
- sin capital universal.

B. paper_portfolio_track_config

- configured_capital;
- currency;
- allocation_policy;
- effective_from/effective_to;
- supersesión;
- confirmación requerida.

C. paper_allocation_request

- requested_notional;
- status;
- requested_at;
- source_log_event_id;
- requested_by;
- resolution.

D. paper_composition_version

- cartera;
- versión;
- effective_at;
- effective_session;
- policy;
- scale_factor;
- configured_capital;
- total_requested;
- total_allocated;
- cash_after;
- reason;
- confirmed_by;
- confirmed_at;
- content_hash.

E. paper_composition_member

- composition_version_id;
- ticker;
- allocation_request_id;
- episode_id;
- requested_notional;
- allocated_notional;
- target_weight;
- quantity_before;
- quantity_after;
- transaction_required.

F. paper_position_episode

- incorporación económica;
- anchor;
- requested/allocated inicial;
- estado;
- cierre/removal;
- referencias al log y tesis.

G. paper_portfolio_flow

Solo flujos externos.

H. paper_portfolio_transaction

- PURCHASE;
- SALE;
- PARTIAL_SALE;
- LEGACY_REBALANCE_BUY;
- LEGACY_REBALANCE_SELL;
- PRO_RATA_REBALANCE_BUY;
- PRO_RATA_REBALANCE_SELL;
- DIVIDEND;
- COMMISSION;
- SPLIT_ADJUSTMENT.

I. paper_track_snapshot

- metodología y composición vigentes;
- cash;
- market value;
- exposure;
- units;
- unit value;
- benchmark;
- cobertura.

Evitar sobrescribir composición o cantidades históricas.

==================================================
11. PRUEBAS LAB
==================================================

Crear o actualizar suites para:

A. REPLAY COMPLETO

- los 378 eventos quedan clasificados;
- ningún evento desaparece;
- orden determinista.

B. 46 → 17

- explicar cada thesis_added;
- reconciliar holdings actuales;
- unresolved explícitos.

C. REBALANCEO SECUENCIAL

- 10 posiciones = 1.000;
- posición 11 = rebalanceo a 909,09;
- posición 12 = rebalanceo a 833,33;
- no reescribir sesiones anteriores.

D. NOTIONAL DESIGUAL

- TARGET_NOTIONAL_PRO_RATA conserva proporciones;
- LEGACY_EQUAL_WEIGHT no.

E. CAMBIO DE POLÍTICA

- no cambia unidades;
- no crea flujo externo;
- mantiene continuidad del unit value.

F. REQUEST SIN EPISODIO

- pending/descartado no crea episodio.

G. REMOVAL SIN PRECIO

- cierra asignación;
- exige cobertura EOD;
- no inventa precio.

H. AISLAMIENTO

- cero cambios de Producción;
- cero cambios económicos;
- cero schema changes.

FAIL = 0
NO_RESULT = 0

==================================================
12. ENTREGA
==================================================

CAT_COMPOSICION_LOG_EVENTS = <número>
CAT_COMPOSICION_EVENT_TYPES = <tabla>
CAT_COMPOSICION_EVENTS_CLASSIFIED = <número>
CAT_COMPOSICION_EVENTS_UNRESOLVED = <número>

PAPER_HISTORICAL_ALLOCATION_REQUESTS = <número>
PAPER_EFFECTIVE_EPISODES = <número>
PAPER_CURRENT_OPEN_EPISODES = <número>
PAPER_CLOSED_OR_REMOVED_EPISODES = <número>
PAPER_NEVER_EFFECTIVE_REQUESTS = <número>
PAPER_REINCORPORATIONS = <número>
PAPER_UNRESOLVED_LINEAGE = <número>

CATALIZADA_THESIS_ADDED = 46
CATALIZADA_CURRENT_HOLDINGS = 17
CATALIZADA_46_TO_17_RECONCILED = PASS/FAIL

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

PROSPECTIVE_ALLOCATION_POLICY =
TARGET_NOTIONAL_PRO_RATA

PROSPECTIVE_CONFIRMATION_REQUIRED = PASS/FAIL
REQUESTED_NOTIONAL_PRESERVED = PASS/FAIL
ALLOCATED_NOTIONAL_VERSIONED = PASS/FAIL
RETROACTIVE_REWRITE_PROHIBITED = PASS/FAIL

POSITION_11_HISTORICAL_BEHAVIOR = <descripción>
POSITION_25_HISTORICAL_BEHAVIOR = <descripción>
UNEQUAL_NOTIONAL_PRO_RATA_VALIDATED = PASS/FAIL

V27_EVENT_REPLAY_SUPPORTED = PASS/FAIL
V27_COMPOSITION_VERSIONING_SUPPORTED = PASS/FAIL
V27_SCHEMA_FINAL_PROPOSAL = <ruta>

PROPOSED_RETURN_METHOD = PRICE_RETURN_TWR
PROPOSED_EXIT_POLICY = KEEP_AS_PAPER_CASH
PROPOSED_DIVIDEND_POLICY = INCOME_NOT_INCLUDED
PROPOSED_COMMISSION_POLICY = NOT_MODELED
PROPOSED_BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

NO_PRODUCTION_SCHEMA_CHANGE = PASS/FAIL
NO_BACKFILL_EXECUTED = PASS/FAIL
NO_PAPER_HISTORY_MUTATED = PASS/FAIL
NO_REAL_DATA_MUTATED = PASS/FAIL
NO_ECONOMIC_WRITES = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

EXACT_BLOCKERS_BEFORE_GATE_2B:
1. <bloqueo>
2. <bloqueo>
3. <bloqueo>

Detenerse.