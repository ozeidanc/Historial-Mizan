MIZAN · LA LENTE
GATE 2A-BIS · TOPOLOGÍA DE CAPITAL, ASIGNACIÓN Y CAPACIDAD
AUDITORÍA + SIMULACIÓN LAB + METODOLOGÍA PREVIA A v27

NATURALEZA

Esta ejecución es exclusivamente:

- auditoría de código y datos;
- reconstrucción point-in-time;
- simulación LAB;
- comparación de políticas;
- propuesta corregida de metodología y schema.

No autoriza:

- migración v27 en Producción;
- metodología SEALED;
- creación de tablas productivas;
- creación de episodios;
- backfill;
- fetch masivo de anchors;
- cambio de UI;
- cambio de daily-close;
- deploy;
- commit;
- tag;
- push.

Detenerse y esperar aprobación de Omar.

==================================================
0. CONTEXTO
==================================================

Baseline:

CANONICAL_HEAD = f30aa3b
SCHEMA_PRODUCTION = v26
single Git root = true
env-info canonical = true

Gate 2A confirmado:

- 5 carteras papel;
- 82 incorporaciones únicas;
- entrada_ts UTC exacto;
- 0 lineage ambiguos;
- 28 correcciones de sesión America/New_York;
- 80 anchors pendientes de cobertura;
- objetivo propuesto = PRICE_RETURN_TWR;
- dividendos todavía no incluidos;
- comisiones no modeladas.

ACLARACIÓN FUNCIONAL DE OMAR

La Lente permite:

1. definir el tamaño de una cartera papel;
2. indicar manualmente el importe deseado al incorporar cada stock;
3. añadir stocks sucesivamente.

Ejemplo que debe auditarse:

portfolio_capital = 10.000 USD
requested_notional_per_stock = 1.000 USD
stocks_added = 25
total_requested = 25.000 USD

Omar desconoce qué hace actualmente Mizan después de que las primeras diez
posiciones consuman los 10.000 USD.

No asumir que:

- todas las carteras son equal-weight;
- el tamaño de todas las carteras es 10.000;
- cada incorporación es una aportación externa;
- cada importe almacenado fue calculado como capital/N;
- Mizan repondera automáticamente;
- Mizan deja posiciones pendientes;
- Mizan permite cash negativo;
- Mizan amplía implícitamente el capital;
- las cantidades actuales representan la intención original de Omar.

==================================================
1. RESTRICCIONES
==================================================

No modificar en Producción:

- schema;
- holdings;
- snapshots;
- valuations;
- tesis;
- catalizadores;
- Campo de Caza;
- carteras papel;
- capital configurado;
- cantidades;
- importes;
- precios;
- Book real;
- cash real;
- NAV real;
- Track de Crecimiento;
- decisiones;
- movimientos;
- resizing.

No crear:

- aportaciones papel;
- retiradas papel;
- rebalanceos;
- compras o ventas papel;
- episodios;
- flujos;
- snapshots unitizados.

No:

- hacer push;
- crear tag;
- modificar remotos;
- imprimir secretos;
- usar Producción como destino de pruebas.

Todas las simulaciones deben ejecutarse en:

- memoria;
- BD LAB;
- o copia temporal aislada.

==================================================
2. AUDITORÍA DEL CONTRATO ACTUAL DE LA UI
==================================================

Localizar con archivos, funciones, endpoints y tablas:

- dónde se define el tamaño de la cartera;
- cómo se llama ese campo en UI;
- cómo se envía al backend;
- cómo se persiste;
- si representa:
  - capital total;
  - capital objetivo;
  - exposición máxima;
  - capital inicial;
  - simple dato informativo;
- dónde se introduce el importe de cada stock;
- cómo se persiste ese importe;
- si representa:
  - requested_notional;
  - allocated_notional;
  - coste;
  - target weight;
  - aportación externa;
  - valor indicativo;
- dónde se calcula quantity;
- qué precio se utiliza;
- cuándo se recalculan pesos;
- qué ocurre al superar el capital.

Auditar expresamente:

- componente UI;
- validaciones frontend;
- validaciones backend;
- endpoint de creación de cartera papel;
- endpoint de incorporación de stock;
- código de snapshots;
- cálculo de cash_pct;
- cálculo de importe;
- cálculo de pesos;
- cualquier normalización automática;
- cualquier rebalanceo;
- cualquier ampliación implícita de capital.

Entregar la cadena exacta:

input de Omar
→ payload
→ validación
→ persistencia
→ quantity
→ importe
→ peso
→ snapshot
→ gráfica.

==================================================
3. CAMPOS ECONÓMICOS QUE DEBEN DISTINGUIRSE
==================================================

Evaluar el contrato futuro separando:

portfolio_capital

Capital externo total disponible para la cartera.

requested_notional

Importe solicitado por Omar para una incorporación.

allocated_notional

Importe que Mizan realmente asigna después de aplicar capacidad y política.

available_paper_cash

Capital disponible antes de incorporar la posición.

paper_market_value

Valor de las posiciones activas.

gross_exposure

Exposición bruta total.

target_weight

Peso objetivo, solo si existe una política de ponderación.

actual_weight

Peso real en la sesión valorada.

allocation_policy

Política utilizada para convertir requested_notional en allocated_notional.

allocation_status

Resultado de la solicitud.

No utilizar un único campo `importe` para representar simultáneamente:

- solicitud;
- asignación;
- capital externo;
- coste;
- peso;
- valor actual.

==================================================
4. POLÍTICAS POSIBLES DE ASIGNACIÓN
==================================================

Identificar qué política aplica hoy Mizan.

Clasificación:

A. MANUAL_NOTIONAL_HARD_CAP

- cada stock intenta recibir requested_notional;
- se financia con paper_cash;
- no se modifican posiciones existentes;
- no se supera portfolio_capital;
- el excedente queda pendiente o se rechaza.

B. PARTIAL_ALLOCATION_TO_AVAILABLE_CASH

- si requested_notional supera el cash disponible;
- se asigna únicamente el cash restante;
- debe existir confirmación explícita o una regla configurada;
- se registra requested_notional distinto de allocated_notional.

C. PRO_RATA_NORMALIZATION

- cuando la suma solicitada supera el capital;
- todas las posiciones se normalizan proporcionalmente;
- puede afectar posiciones existentes.

D. EXPLICIT_EQUAL_WEIGHT_REBALANCE

- todas las posiciones pasan a capital/N;
- requiere un rebalanceo fechado;
- genera compras y ventas internas point-in-time;
- no puede reescribir cantidades o anchors históricos.

E. IMPLICIT_CAPITAL_EXPANSION

- Mizan aumenta el capital para aceptar nuevas posiciones;
- equivale a una aportación externa;
- solo es válido con evento explícito.

F. LEVERAGED_OR_NEGATIVE_CASH

- la exposición puede superar el capital;
- requiere política de apalancamiento explícita;
- no debe permitirse silenciosamente.

G. OTHER

- explicar exactamente.

No seleccionar todavía la política futura.

Primero identificar:

CURRENT_ALLOCATION_POLICY

y:

CURRENT_OVER_CAPACITY_BEHAVIOR.

==================================================
5. CASO OBLIGATORIO 25 × 1.000 SOBRE 10.000
==================================================

Reproducir en LAB el caso:

portfolio_capital = 10.000 USD
requested_notional = 1.000 USD por stock
25 incorporaciones sucesivas
total_requested = 25.000 USD

Mostrar después de cada incorporación:

- stock sequence;
- requested_notional;
- allocated_notional;
- paper_cash_before;
- paper_cash_after;
- capital_before;
- capital_after;
- gross_exposure;
- position count;
- existing positions changed;
- quantities changed;
- weights changed;
- external flow created;
- allocation_status;
- warning shown to Omar.

Determinar qué ocurre:

- tras la posición 10;
- en la posición 11;
- entre las posiciones 11 y 25;
- al reconstruir snapshots;
- al abrir posteriormente la cartera;
- al calcular la gráfica.

Verificar si las primeras diez posiciones son reponderadas retroactivamente
cuando se incorporan las quince posteriores.

Está prohibido tratar una reponderación posterior como si hubiera existido
desde el inicio.

Si se detecta reponderación:

- identificar la sesión y hora;
- reconstruir las compras y ventas internas;
- conservar la composición anterior;
- no reescribir anchors iniciales;
- tratarlo como rebalanceo point-in-time.

==================================================
6. AUDITORÍA DE LAS CINCO CARTERAS EXISTENTES
==================================================

Para cada cartera entregar:

- paper_portfolio_id;
- nombre;
- created_at;
- configured_portfolio_size;
- configured_currency;
- target_position_count, si existe;
- allocation mode seleccionado, si existe;
- number_of_incorporations;
- first_incorporation;
- last_incorporation;
- requested_notional por posición;
- stored importe por posición;
- original quantity;
- original entry price;
- suma de importes;
- cash_pct;
- cash implícito;
- exposición implícita;
- peso por posición;
- cambios de peso entre snapshots;
- señales de normalización;
- señales de rebalanceo;
- señales de aportación implícita;
- señales de cash negativo.

Determinar si los importes:

588,24
400
714,29
10.000

proceden realmente de:

- una elección manual de Omar;
- capital/N;
- normalización automática;
- configuración de número objetivo de posiciones;
- otro cálculo.

No inferir equal-weight solo porque los importes coinciden.

Localizar la función exacta que produjo esos importes.

Declarar por cartera:

PORTFOLIO_CAPITAL_PROVEN
MANUAL_NOTIONAL_SUPPORTED
EQUAL_WEIGHT_EXPLICIT
EQUAL_WEIGHT_INFERRED
NORMALIZATION_DETECTED
REBALANCE_DETECTED
IMPLICIT_CONTRIBUTION_DETECTED
NEGATIVE_CASH_DETECTED
OVER_CAPACITY_POSITIONS

==================================================
7. RECONSTRUCCIÓN POINT-IN-TIME DE ASIGNACIONES
==================================================

Para cada una de las 82 incorporaciones proponer:

- requested_notional;
- source_of_requested_notional;
- allocated_notional;
- source_of_allocated_notional;
- capital_available_before;
- paper_cash_before;
- allocation_policy_applied;
- allocation_status;
- external_flow;
- internal_transaction;
- paper_cash_after;
- cumulative_allocated_notional;
- gross_exposure_after;
- existing_positions_rebalanced;
- rebalance_event_required;
- confidence;
- warning.

Valores de confidence:

- PROVEN_FROM_INPUT;
- PROVEN_FROM_CODE_PATH;
- PROVEN_FROM_SNAPSHOT;
- DETERMINISTIC_RECONSTRUCTION;
- INFERRED;
- UNKNOWN.

No declarar automáticamente:

certified_notional = 10.000 / N.

No rederivar automáticamente quantity hasta conocer el importe realmente
asignado y la política aplicada.

==================================================
8. ESTADOS DE CAPACIDAD PROPUESTOS
==================================================

Evaluar estos estados:

FULLY_ALLOCATED

allocated_notional = requested_notional.

PARTIALLY_ALLOCATED

0 < allocated_notional < requested_notional.

PENDING_CAPITAL

La solicitud se conserva, pero no existe capital suficiente.

REJECTED_CAPACITY

No se crea posición por superar la capacidad.

REBALANCE_REQUIRED

La solicitud solo puede satisfacerse mediante un rebalanceo explícito.

EXTERNAL_CONTRIBUTION_REQUIRED

La solicitud requiere una aportación externa explícita.

LEVERAGE_REQUIRED

La solicitud exige exposición superior al capital disponible.

No convertir automáticamente:

PENDING_CAPITAL
→ posición activa.

No crear automáticamente:

- aportación;
- leverage;
- rebalanceo;
- reducción de posiciones existentes.

==================================================
9. POLÍTICA FUTURA RECOMENDADA A SIMULAR
==================================================

Simular como política segura por defecto:

allocation_policy =
MANUAL_NOTIONAL_HARD_CAP

Reglas:

1. Cada cartera tiene un portfolio_capital configurable.
2. Omar introduce requested_notional para cada stock.
3. La compra se financia con paper_cash.
4. Si paper_cash >= requested_notional:
   - allocated_notional = requested_notional;
   - allocation_status = FULLY_ALLOCATED;
   - no se emiten unidades;
   - no existe flujo externo.
5. Si 0 < paper_cash < requested_notional:
   - no asignar parcialmente sin permiso;
   - allocation_status = PENDING_CAPITAL o REBALANCE_REQUIRED;
   - ofrecer opciones explícitas.
6. Si paper_cash = 0:
   - no crear posición activa;
   - no permitir cash negativo;
   - no ampliar capital;
   - no reponderar automáticamente.
7. Un rebalanceo requiere una acción explícita de Omar.
8. Una aportación requiere una acción explícita de Omar.
9. El apalancamiento queda desactivado por defecto.

Opciones que la UI podría ofrecer al superar capacidad:

A. Mantener pendiente.
B. Asignar solo cash disponible.
C. Rebalancear explícitamente.
D. Crear aportación externa explícita.
E. Cancelar incorporación.

No implementar todavía la UI.

Comparar esta política con el comportamiento actual.

==================================================
10. REBALANCEOS
==================================================

Si Omar solicita reponderar:

- crear evento de rebalanceo paper explícito;
- fijar effective_session;
- valorar posiciones con cierres oficiales;
- registrar ventas y compras internas;
- preservar episodios;
- no cambiar incorporated_at;
- no cambiar anchor_session;
- no cambiar catalyst snapshot original;
- no reescribir el Track anterior.

Ejemplo:

capital = 10.000
10 posiciones de 1.000
posteriormente 25 posiciones objetivo

Si Omar ordena equal-weight:

target_notional aproximado = 400 por posición

Pero debe registrarse en la fecha del rebalanceo:

- ventas parciales de las diez existentes;
- compras/asignaciones de las quince nuevas;
- cantidades antes;
- cantidades después;
- precios point-in-time;
- paper cash;
- cero flujo externo;
- cero cambio de unidades por el rebalanceo;
- impacto económico únicamente por precios/costes soportados.

No recalcular la historia anterior como 25 × 400 desde el comienzo.

==================================================
11. CAPITAL Y UNITIZACIÓN
==================================================

No guardar initial_capital_amount como propiedad universal de la metodología.

Separar:

A. METODOLOGÍA

Define reglas:

- PRICE_RETURN_TWR;
- anchors;
- flujo externo;
- operaciones internas;
- dividend policy;
- commission policy;
- benchmark policy;
- allocation policy permitida.

B. CONFIGURACIÓN POR CARTERA

Debe contener:

- paper_portfolio_id;
- methodology_id;
- configured_capital;
- currency;
- allocation_policy;
- partial_allocation_policy;
- leverage_allowed;
- target_position_count, si existe;
- inception_session;
- status;
- configuration_version.

Cada cartera puede tener un capital diferente.

Una modificación futura de configured_capital debe distinguir:

- corrección de configuración antes de empezar;
- aportación externa posterior;
- reducción/retirada de capital;
- nueva versión de configuración.

No modificar capital histórico in-place después de comenzar.

==================================================
12. FLUJOS EXTERNOS Y OPERACIONES INTERNAS
==================================================

FLUJOS EXTERNOS

Cambian capital y unidades:

- PAPER_INITIAL_CONTRIBUTION;
- PAPER_CAPITAL_CONTRIBUTION;
- PAPER_CAPITAL_WITHDRAWAL;
- PAPER_CAPITAL_CORRECTION, con clasificación.

OPERACIONES INTERNAS

No cambian unidades:

- PAPER_POSITION_PURCHASE;
- PAPER_POSITION_SALE;
- PAPER_PARTIAL_SALE;
- PAPER_REBALANCE_BUY;
- PAPER_REBALANCE_SELL;
- PAPER_DIVIDEND;
- PAPER_COMMISSION;
- PAPER_SPLIT_ADJUSTMENT.

Una incorporación solo crea PAPER_POSITION_PURCHASE si fue realmente
asignada.

Una solicitud PENDING_CAPITAL no crea:

- posición activa;
- compra;
- episodio económico efectivo;
- exposición.

Puede conservarse como solicitud pendiente separada.

==================================================
13. BENCHMARK
==================================================

Mantener como propuesta:

SPY_BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

El benchmark replica:

- capital inicial real de cada cartera;
- aportaciones externas;
- retiradas externas.

No replica:

- compras internas;
- ventas internas;
- rebalanceos;
- solicitudes pendientes;
- cambios de peso internos.

Si la cartera mantiene paper cash:

- SPY permanece invertido;
- la comparación incluye la decisión activa de no invertir todo el capital.

No asumir que todas las carteras reciben 10.000.

SPY recibe el configured_capital específico de cada cartera.

==================================================
14. PROPUESTA CORREGIDA DE SCHEMA v27
==================================================

Proponer, sin aplicar, las siguientes responsabilidades.

A. paper_track_methodology

No contener un initial_capital_amount universal.

Campos conceptuales:

- methodology_id;
- version;
- return_method;
- anchor_policy;
- price_basis;
- dividend_policy;
- commission_policy;
- benchmark_policy;
- market_timezone;
- status;
- approved_by;
- approved_at;
- methodology_hash;
- created_at.

B. paper_portfolio_track_config

- config_id;
- paper_portfolio_id;
- methodology_id;
- configured_capital;
- currency;
- allocation_policy;
- partial_allocation_policy;
- leverage_allowed;
- target_position_count;
- inception_session;
- effective_from;
- effective_to;
- supersedes_config_id;
- status;
- content_hash;
- created_at.

No modificar una configuración histórica in-place.

C. paper_allocation_request

- request_id;
- paper_portfolio_id;
- ticker;
- thesis_id;
- requested_at_utc;
- requested_notional;
- currency;
- status;
- allocated_notional;
- allocation_reason;
- resolved_at;
- episode_id;
- content_hash;
- created_at.

D. paper_position_episode

Además de los campos ya propuestos, incluir:

- allocation_request_id;
- requested_notional;
- allocated_notional;
- allocation_policy;
- allocation_status;
- capital_available_before;
- paper_cash_before;
- paper_cash_after;
- original_quantity;
- certified_quantity;
- original_entry_price;
- anchor_price.

E. paper_portfolio_flow

Solo flujos externos.

F. paper_portfolio_transaction

Operaciones internas:

- purchase;
- sale;
- partial sale;
- rebalance buy;
- rebalance sell;
- dividend;
- commission;
- split adjustment.

G. paper_track_snapshot

Incluir:

- configured_capital;
- cumulative_external_contributions;
- cumulative_external_withdrawals;
- paper_cash;
- paper_market_value;
- gross_exposure;
- pending_requested_notional;
- units;
- unit_value;
- retorno;
- benchmark;
- cobertura.

La propuesta puede simplificarse si el schema existente ya conserva de forma
fiable alguna de estas responsabilidades.

No duplicar tablas sin necesidad.

==================================================
15. SIMULACIONES LAB
==================================================

Ejecutar al menos:

CASO A · 10.000 / 10 × 1.000

Resultado esperado bajo MANUAL_NOTIONAL_HARD_CAP:

- 10 posiciones activas;
- 10.000 asignados;
- cash = 0;
- cero flujo externo adicional;
- cero leverage.

CASO B · POSICIÓN 11 DE 1.000

- no crear posición activa automáticamente;
- status = PENDING_CAPITAL o REBALANCE_REQUIRED;
- cash = 0;
- capital sigue 10.000;
- exposición no aumenta.

CASO C · 25 × 1.000

- primeras 10 según orden de aceptación;
- 15 pendientes;
- total solicitado = 25.000;
- total asignado = 10.000;
- pending requested = 15.000;
- sin cash negativo;
- sin aportación implícita.

CASO D · ASIGNACIÓN PARCIAL

- cash = 400;
- solicitud = 1.000;
- no asignar 400 sin política o confirmación explícita.

CASO E · APORTACIÓN EXPLÍCITA

- Omar aporta 1.000;
- aumenta cash;
- emite unidades;
- no genera retorno;
- permite resolver una solicitud pendiente.

CASO F · REBALANCEO EXPLÍCITO

- 10 posiciones de 1.000;
- 15 pendientes;
- Omar ordena reponderar 25;
- registrar rebalanceo en la sesión;
- target aproximado 400;
- no reescribir historia anterior;
- unidades no cambian;
- flujo externo = 0.

CASO G · CAPITAL CONFIGURABLE

- cartera A = 10.000;
- cartera B = 25.000;
- misma metodología;
- distinta configuración y capacidad.

CASO H · IMPORTE MANUAL NO EQUAL-WEIGHT

- solicitudes 2.000, 500 y 1.250;
- respetar importes;
- no normalizar sin orden.

CASO I · TRACK TWR

- compras internas no cambian unit value;
- aportación externa emite unidades;
- rebalanceo no emite unidades.

CASO J · BENCHMARK

- replica solo flujos externos;
- no replica compras o rebalanceos.

FAIL = 0
NO_RESULT = 0

==================================================
16. ENTREGA
==================================================

CURRENT_PORTFOLIO_SIZE_FIELD = <campo/ruta>
CURRENT_MANUAL_NOTIONAL_FIELD = <campo/ruta>
CURRENT_QUANTITY_CALCULATION = <función>
CURRENT_WEIGHT_CALCULATION = <función>

CURRENT_ALLOCATION_POLICY =
MANUAL_NOTIONAL_HARD_CAP /
PARTIAL_ALLOCATION /
PRO_RATA_NORMALIZATION /
EXPLICIT_EQUAL_WEIGHT /
IMPLICIT_CAPITAL_EXPANSION /
LEVERAGED_OR_NEGATIVE_CASH /
OTHER /
UNDETERMINED

CURRENT_OVER_CAPACITY_BEHAVIOR = <descripción>

CURRENT_POSITION_11_BEHAVIOR = <descripción>
CURRENT_POSITION_25_BEHAVIOR = <descripción>

PAPER_PORTFOLIO_CAPITAL_PROVEN = PASS/FAIL
MANUAL_NOTIONAL_SUPPORTED = PASS/FAIL
EXISTING_EQUAL_WEIGHT_WAS_EXPLICIT = PASS/FAIL
NORMALIZATION_DETECTED = PASS/FAIL
RETROACTIVE_REWEIGHTING_DETECTED = PASS/FAIL
IMPLICIT_CONTRIBUTION_DETECTED = PASS/FAIL
NEGATIVE_CASH_DETECTED = PASS/FAIL
LEVERAGE_DETECTED = PASS/FAIL

Por cartera:

- configured capital;
- sum requested notional;
- sum allocated notional;
- implied cash;
- gross exposure;
- over-capacity requests;
- allocation model confidence.

PROPOSED_DEFAULT_ALLOCATION_POLICY =
MANUAL_NOTIONAL_HARD_CAP

PROPOSED_PARTIAL_ALLOCATION_POLICY =
REQUIRE_EXPLICIT_CONFIRMATION

PROPOSED_LEVERAGE_POLICY =
DISABLED

PROPOSED_OVER_CAPACITY_STATUS =
PENDING_CAPITAL

PROPOSED_RETURN_METHOD =
PRICE_RETURN_TWR

PROPOSED_EXIT_POLICY =
KEEP_AS_PAPER_CASH

PROPOSED_DIVIDEND_POLICY =
INCOME_NOT_INCLUDED

PROPOSED_COMMISSION_POLICY =
NOT_MODELED

PROPOSED_BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

V27_SCHEMA_REVISED = PASS/FAIL
INITIAL_CAPITAL_REMOVED_FROM_GLOBAL_METHODOLOGY = PASS/FAIL
PORTFOLIO_SPECIFIC_CAPITAL_MODELED = PASS/FAIL
REQUESTED_AND_ALLOCATED_NOTIONAL_SEPARATED = PASS/FAIL
REBALANCE_POINT_IN_TIME_MODELED = PASS/FAIL

NO_PRODUCTION_SCHEMA_CHANGE = PASS/FAIL
NO_BACKFILL_EXECUTED = PASS/FAIL
NO_PAPER_HISTORY_MUTATED = PASS/FAIL
NO_REAL_DATA_MUTATED = PASS/FAIL
NO_ECONOMIC_WRITES = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

EXACT_DECISIONS_REQUIRED_FROM_OMAR:
1. <decisión>
2. <decisión>
3. <decisión>

Detenerse.