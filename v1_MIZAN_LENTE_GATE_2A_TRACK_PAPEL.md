MIZAN · LA LENTE
GATE 2A · DISEÑO Y PREVIEW DEL TRACK RECORD PAPEL UNITIZADO
IDENTIDAD DE EPISODIOS · ANCHORS EOD · FLUJOS · BENCHMARK

NATURALEZA DE ESTA EJECUCIÓN

Este Gate 2A es exclusivamente:

- auditoría;
- diseño;
- simulación en LAB;
- clasificación de registros históricos;
- preview de anchors;
- comparación de metodologías;
- propuesta de migración.

No autoriza todavía:

- migrar Producción a v27;
- crear episodios en Producción;
- crear flujos papel en Producción;
- ejecutar backfill;
- persistir snapshots papel nuevos;
- cambiar daily-close;
- cambiar endpoints productivos;
- cambiar la UI;
- desplegar;
- crear tags;
- hacer push.

Detenerse al terminar el Gate 2A y esperar decisiones expresas de Omar.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = f30aa3b
schema de Producción = v26
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Hallazgos aceptados del Gate 1:

- existen 5 carteras papel;
- existen 82 registros históricos de holdings en snapshots cat:*;
- esos 82 registros NO se consideran automáticamente 82 episodios únicos;
- holdings.entrada_ts es epoch-ms UTC exacto;
- 0 timestamps de incorporación ambiguos;
- 0 registros DATE_ONLY para entrada_ts;
- snapshots.fecha utiliza fecha UTC;
- 28 de 82 registros tienen fecha UTC distinta de la fecha de mercado NY;
- daily-close no persiste una curva diaria completa para cat:*;
- el anchor actual puede usar precio vivo o apertura en lugar de cierre oficial;
- SPY no está correctamente anclado point-in-time;
- la UI Papel utiliza un wrapper distinto al Track de Crecimiento;
- las carteras papel están excluidas de Book, cash y NAV real;
- el Track de Crecimiento usa retorno sobre coste, no TWR unitizado;
- el objetivo metodológico propuesto para Papel es una serie unitizada.

Objetivo metodológico sujeto a validación:

PROPOSED_PAPER_RETURN_METHOD = UNITIZED_TWR

No declarar UNITIZED_TWR como método certificado hasta resolver:

1. identidad de incorporaciones únicas;
2. notional histórico de cada flujo;
3. política de capital después de una salida;
4. tratamiento de dividendos;
5. tratamiento de comisiones;
6. benchmark SPY con los mismos flujos externos;
7. cobertura completa de anchors y precios.

==================================================
1. RESTRICCIONES ABSOLUTAS
==================================================

No modificar en Producción:

- schema;
- SQLite;
- holdings;
- snapshots;
- valuations;
- tesis;
- catalizadores;
- Campo de Caza;
- Book real;
- cash real;
- NAV real;
- Track de Crecimiento;
- movimientos;
- decisiones;
- resizing;
- posiciones reales;
- Defensiva;
- Equilibrada;
- revisión diaria;
- selector;
- política.

No crear en Producción:

- paper_position_episode;
- paper_portfolio_flow;
- paper_track_snapshot;
- episodios;
- flujos;
- anchors certificados;
- snapshots;
- backfill;
- decisiones o movimientos.

No:

- hacer push;
- crear o modificar remotos;
- hacer fetch o pull;
- crear tags;
- conectar con Wio;
- imprimir secretos;
- borrar los tres backups HTML untracked;
- alterar la historia papel original.

Todas las simulaciones deben ejecutarse:

- en memoria;
- en una base LAB aislada;
- o sobre una copia temporal de la base.

No usar la base de Producción como destino de escritura.

Los datos productivos solo pueden consultarse en modo read-only.

Si una herramienta o prueba no puede garantizar aislamiento:

- no ejecutarla;
- reportar el bloqueo;
- no sustituirlo por una escritura en Producción.

==================================================
2. VERIFICACIÓN PREVIA
==================================================

Confirmar read-only:

- CANONICAL_HEAD;
- env-info;
- raíz Git única;
- git status;
- schema v26;
- integrity_check;
- health;
- una sola instancia;
- contadores económicos;
- contadores papel;
- 5 carteras papel;
- 82 registros históricos;
- tres backups HTML untracked.

Capturar para comprobación posterior:

- movimientos reales;
- cash events;
- baselines;
- decisiones;
- resizing;
- holdings papel;
- snapshots papel;
- tesis;
- catalizadores;
- valuations reales.

No limpiar ni modificar el working tree.

Declarar:

LAB_ISOLATED_FROM_PRODUCTION = PASS/FAIL
PRODUCTION_READ_ONLY = PASS/FAIL

Si alguno falla:

- detenerse;
- no continuar con la simulación.

==================================================
3. IDENTIDAD DE INCORPORACIONES ÚNICAS
==================================================

No asumir:

PAPER_POSITION_RECORDS = PAPER_EPISODES

Clasificar cada uno de los 82 registros usando toda la evidencia disponible:

- paper_portfolio_id;
- cartera cat:*;
- ticker;
- source_holding_id, si existe;
- source_snapshot_id;
- entrada_ts;
- precio_entrada;
- quantity;
- thesis_id;
- catalyst_snapshot;
- fecha_salida;
- precio_salida;
- status;
- lineage disponible.

Clasificación obligatoria:

A. UNIQUE_INCORPORATION

Existe evidencia de una incorporación única a una cartera papel.

B. REPEATED_SNAPSHOT_OF_SAME_INCORPORATION

La fila representa la misma incorporación ya presente en otro snapshot.

C. REINCORPORATION

El ticker tuvo un episodio anterior cerrado y posteriormente una nueva
incorporación demostrable mediante un nuevo entrada_ts o evento independiente.

D. AMBIGUOUS_LINEAGE

No puede determinarse de forma segura si es una incorporación, una repetición
o una reincorporación.

Una identidad propuesta de episodio debe utilizar, cuando sea fiable:

- paper_portfolio_id;
- ticker;
- entrada_ts;
- source_holding_id;
- thesis_id.

No utilizar snapshot_id como identidad única si un mismo holding aparece en
múltiples snapshots.

No deduplicar únicamente por ticker.

El mismo ticker puede tener varios episodios si existen reincorporaciones.

Entregar:

PAPER_HOLDING_ROWS
PAPER_UNIQUE_INCORPORATIONS
PAPER_REPEATED_SNAPSHOT_ROWS
PAPER_REINCORPORATIONS
PAPER_AMBIGUOUS_LINEAGE

Para cada AMBIGUOUS_LINEAGE:

- explicar la evidencia disponible;
- presentar interpretaciones posibles;
- no proponer backfill automático.

==================================================
4. EVENTO TEMPORAL CANÓNICO
==================================================

Fuente temporal canónica:

holdings.entrada_ts

No utilizar como sustituto:

- snapshots.fecha;
- tesis.creado_en;
- tesis.sellada_en;
- fecha del proceso;
- fecha mostrada en Dubái;
- timezone del navegador;
- precio de entrada como evidencia temporal.

Cadena temporal:

entrada_ts epoch-ms UTC
→ America/New_York mediante zona IANA
→ calendario NYSE
→ market_state
→ anchor_session
→ anchor_available_at
→ episode_effective_at

Zona informativa del usuario:

Asia/Dubai

Zona económica del mercado:

America/New_York

No aplicar una resta fija de 8 o 9 horas.

La hora de Dubái debe mostrarse en el preview únicamente para que Omar pueda
reconocer el momento de incorporación.

La sesión económica debe resolverse siempre en America/New_York.

Estados:

- PRE_MARKET;
- REGULAR_OPEN;
- AFTER_CLOSE;
- NON_TRADING_DAY.

Reglas propuestas:

A. REGULAR_OPEN

- anchor_session = sesión en curso;
- anchor_price = cierre oficial de esa sesión;
- anchor_available_at = momento posterior al cierre en que el precio quedó
  disponible;
- episode_effective_at = cierre de la sesión;
- retorno inicial = 0;
- primera variación = siguiente sesión valorada.

No atribuir rendimiento desde entrada_ts hasta el cierre.

B. PRE_MARKET

- anchor_session = última sesión completamente cerrada;
- episode_effective_at = instante de incorporación.

C. AFTER_CLOSE

- anchor_session = sesión que acaba de cerrar;
- episode_effective_at = instante de incorporación.

D. NON_TRADING_DAY

- anchor_session = última sesión completamente cerrada;
- episode_effective_at = instante de incorporación.

Comprobar especialmente los 28 registros en los que:

snapshots.fecha UTC ≠ market_date America/New_York

No modificar snapshots.fecha.

Persistir en el futuro la sesión correcta en una entidad nueva.

==================================================
5. PREVIEW DE ANCHORS
==================================================

Generar un preview read-only para cada incorporación única propuesta.

Campos mínimos:

- proposed_episode_id;
- classification;
- paper_portfolio_id;
- portfolio_name;
- ticker;
- source_holding_id;
- source_snapshot_ids;
- thesis_id;
- entrada_ts_epoch;
- incorporated_at_utc;
- incorporated_at_dubai;
- incorporated_at_new_york;
- market_date_new_york;
- market_state;
- snapshots_fecha_original;
- current_entry_price;
- current_entry_price_source;
- previous_closed_session;
- incorporation_session;
- proposed_anchor_session;
- proposed_anchor_price;
- proposed_anchor_price_source;
- anchor_price_basis;
- anchor_available_at;
- proposed_episode_effective_at;
- quantity;
- original_recorded_notional;
- certified_anchor_notional;
- session_changed_by_utc_correction;
- price_coverage_status;
- SPY_price_coverage_status;
- lineage_status;
- automatic_backfill_eligibility;
- warning;
- resolution_reason.

Contrato del anchor propuesto:

anchor_price_type = PAPER_REFERENCE_CLOSE
execution_status = NOT_EXECUTED
market_timezone = America/New_York
price_basis = OFFICIAL_CLOSE_NOMINAL

No presentar el anchor como una ejecución real.

No utilizar fallback a:

- precio vivo;
- apertura;
- precio de La Lente;
- precio actual;
- cierre de otra sesión;
- SPY actual.

Si falta el cierre:

price_coverage_status = PRICE_COVERAGE_INCOMPLETE

No inventar el precio.

Clasificar cada incorporación:

- AUTO_BACKFILL_SAFE;
- MANUAL_REVIEW_REQUIRED;
- PRICE_COVERAGE_INCOMPLETE;
- LINEAGE_AMBIGUOUS.

==================================================
6. COHERENCIA DE PRECIOS
==================================================

Comprobar que el Track Papel futuro pueda usar una base coherente para:

- anchor;
- valoración diaria;
- salida;
- benchmark.

Base esperada:

OFFICIAL_CLOSE_NOMINAL

Tratamiento esperado:

- mismo soporte de splits que Crecimiento;
- dividendos separados del precio;
- sin adjusted-close total-return silencioso.

No mezclar:

- close nominal con adjusted close;
- precio vivo con cierre;
- apertura con cierre;
- precio actual con anchor histórico.

Auditar para cada ticker:

- cobertura del anchor;
- cobertura entre anchor y última sesión;
- splits;
- dividendos;
- cambios de ticker;
- bajas de cotización;
- huecos de precios.

No corregir datos todavía.

Entregar:

ANCHORS_WITH_FULL_COVERAGE
ANCHORS_PRICE_COVERAGE_INCOMPLETE
EPISODES_WITH_SPLITS
EPISODES_WITH_DIVIDENDS
EPISODES_WITH_PRICE_GAPS

==================================================
7. NOTIONAL HISTÓRICO
==================================================

El modelo unitizado necesita determinar el flujo externo asociado a cada
incorporación.

Evaluar como mínimo:

A. ORIGINAL_RECORDED_COST

original_recorded_notional =
quantity × original_recorded_entry_price

Ventaja:

- refleja el coste que Mizan registró históricamente.

Riesgo:

- el precio original puede ser vivo, apertura o indicativo;
- puede ser incoherente con el anchor EOD certificado.

B. CERTIFIED_ANCHOR_VALUE

certified_anchor_notional =
quantity × certified_anchor_price

Ventaja:

- posición y flujo nacen al mismo cierre;
- P&L inicial = 0;
- coherencia point-in-time.

Riesgo:

- reconstruye un capital distinto del coste registrado originalmente;
- debe etiquetarse como simulación certificada, no como el coste original.

C. FIXED_OR_EQUAL_NOTIONAL

Solo evaluar si existe evidencia de que las carteras catalizadas se
construyeron con un notional fijo o equiponderado.

No inventar esta opción si no existe evidencia.

Mantener siempre separados:

- original_recorded_entry_price;
- original_recorded_notional;
- certified_anchor_price;
- certified_paper_flow_notional.

Simular las opciones deterministas.

Comparar por cartera:

- capital acumulado;
- NAV;
- unit value;
- retorno;
- drawdown;
- benchmark;
- sensibilidad del resultado.

No elegir automáticamente.

Entregar recomendación razonada:

HISTORICAL_NOTIONAL_SOURCE =
ORIGINAL_RECORDED_COST /
CERTIFIED_ANCHOR_VALUE /
FIXED_NOTIONAL /
DECISION_REQUIRED

==================================================
8. MODELO UNITIZADO PROPUESTO
==================================================

Simular en LAB una serie unitizada.

Valor inicial:

paper_unit_value = 100,00

Para cada flujo externo positivo:

units_issued =
external_contribution / pre_flow_unit_value

Para cada flujo externo negativo:

units_redeemed =
external_withdrawal / pre_flow_unit_value

Identidad:

paper_NAV =
paper_market_value + paper_cash

paper_unit_value =
paper_NAV / units_outstanding

daily_return =
paper_unit_value_t / paper_unit_value_t-1 - 1

cumulative_return =
paper_unit_value_t / initial_unit_value - 1

Orden obligatorio en una incorporación al cierre:

1. valorar la cartera existente con cierres de la sesión;
2. calcular pre_flow_NAV;
3. calcular pre_flow_unit_value;
4. registrar la aportación papel;
5. emitir unidades a pre_flow_unit_value;
6. comprar la posición al anchor certificado;
7. registrar post_flow_NAV;
8. comprobar que unit_value no cambia por el flujo;
9. comprobar que la nueva posición empieza con P&L 0.

Persistir conceptualmente en la propuesta:

- pre_flow_NAV;
- pre_flow_unit_value;
- external_flow;
- units_delta;
- post_flow_NAV;
- post_flow_unit_value.

Prueba obligatoria:

- cartera empieza con 100;
- gana 10 %;
- unit value pasa a 110;
- se incorpora una nueva aportación de 100;
- NAV pasa a 210;
- se emiten 100/110 unidades;
- unit value permanece en 110;
- retorno acumulado permanece en 10 %.

No declarar todavía que esta metodología está certificada.

Clasificar al final:

PROPOSED_RETURN_METHOD =
UNITIZED_TWR /
PRICE_RETURN_TWR /
UNDETERMINED

Si dividendos no están soportados:

- no denominarla total-return TWR;
- usar PRICE_RETURN_TWR;
- mostrar INCOME_NOT_INCLUDED.

==================================================
9. POLÍTICA DE SALIDA PENDIENTE
==================================================

No asumir que cerrar una posición implica retirar capital.

Simular dos políticas:

A. KEEP_AS_PAPER_CASH

Al cerrar una posición:

1. reconocer el movimiento de precio hasta exit_price;
2. vender la posición;
3. aumentar paper_cash;
4. mantener las unidades;
5. mantener el capital dentro de la cartera;
6. no registrar flujo externo;
7. unit_value solo cambia por resultado económico, no por la venta.

Las incorporaciones posteriores pueden financiarse con:

- paper_cash disponible;
- nueva aportación externa;
- una combinación explícita.

B. EXTERNAL_WITHDRAWAL

Al cerrar una posición:

1. reconocer el movimiento de precio hasta exit_price;
2. vender la posición;
3. registrar una retirada externa por el importe retirado;
4. retirar unidades a pre_flow_unit_value;
5. no cambiar unit_value por la retirada.

Determinar qué política representa mejor el comportamiento histórico y el
propósito de las carteras catalizadas.

Analizar:

- si el capital de una posición cerrada permanecía conceptualmente en la
  cartera;
- si las nuevas incorporaciones reutilizaban capital anterior;
- si cada episodio era una apuesta independiente;
- si existe cash papel actual;
- si existe evidencia en UI, código o datos.

No seleccionar una política sin aprobación de Omar.

Entregar:

EXIT_CAPITAL_POLICY =
KEEP_AS_PAPER_CASH /
EXTERNAL_WITHDRAWAL /
DECISION_REQUIRED

Mostrar el efecto histórico de ambas opciones:

- NAV;
- unidades;
- retorno;
- drawdown;
- benchmark;
- cash papel.

==================================================
10. DIVIDENDOS, COMISIONES Y SPLITS
==================================================

DIVIDENDOS

Determinar:

- si existen dividendos durante los episodios;
- si Mizan puede atribuirlos a cada posición papel;
- si existe fecha ex-dividend;
- si existe fecha de pago;
- si existe retención;
- si Crecimiento los trata en línea separada.

Evaluar:

A. PAPER_CASH

- el dividendo aumenta paper_cash;
- aumenta NAV;
- aumenta unit_value;
- forma parte del retorno;
- no es flujo externo.

B. REINVEST

- reinversión explícita con regla y precio verificables;
- no asumirla sin evidencia.

C. UNSUPPORTED

- excluirlo;
- etiquetar la serie como PRICE_RETURN_TWR;
- mostrar INCOME_NOT_INCLUDED.

No ignorar dividendos y denominar el resultado total-return.

COMISIONES

Determinar si existen comisiones papel históricas.

Si existen y son fiables:

- reducen paper_cash/NAV;
- reducen unit_value;
- no son flujo externo neutral.

Si no existen:

- no inventarlas;
- declarar NOT_MODELED.

SPLITS

- ajustar cantidades mediante la misma metodología de Crecimiento;
- no generar retorno artificial;
- no cambiar el valor económico por el split.

Entregar:

DIVIDEND_POLICY =
PAPER_CASH /
REINVEST /
UNSUPPORTED /
DECISION_REQUIRED

COMMISSION_POLICY =
REDUCE_NAV /
NOT_MODELED /
DECISION_REQUIRED

SPLIT_HANDLING =
SUPPORTED /
INCOMPLETE /
DECISION_REQUIRED

==================================================
11. BENCHMARK SPY
==================================================

El benchmark debe replicar únicamente los mismos flujos externos que la
cartera papel.

No debe replicar automáticamente cada compra o venta interna si el capital
permanece dentro de la cartera.

Para cada aportación externa papel:

- misma sesión;
- mismo importe;
- cierre oficial SPY;
- emisión de unidades benchmark;
- cero retorno por el flujo.

Para cada retirada externa:

- misma sesión;
- mismo importe externo;
- retirada proporcional de unidades benchmark;
- cero retorno por el flujo.

Si la política de salida es KEEP_AS_PAPER_CASH:

- una venta interna no debe convertirse automáticamente en retirada del
  benchmark;
- evaluar si el benchmark permanece invertido en SPY;
- o si debe mantener un componente benchmark cash equivalente;
- presentar ambas alternativas y su significado.

Si la política es EXTERNAL_WITHDRAWAL:

- replicar la misma retirada externa en el benchmark.

No utilizar:

- SPY actual;
- un único anchor SPY global;
- un SPY de la primera incorporación para todas las entradas.

Simular al menos:

- dos incorporaciones en sesiones diferentes;
- una salida;
- una cartera que mantiene cash;
- una cartera que retira capital.

Entregar:

SPY_BENCHMARK_FEASIBLE = PASS/FAIL
SPY_BENCHMARK_COVERAGE = <porcentaje>
SPY_EXTERNAL_FLOW_MATCHING = PASS/FAIL
SPY_EXIT_POLICY = <descripción/DECISION_REQUIRED>

==================================================
12. SIMULACIÓN HISTÓRICA EN LAB
==================================================

Reconstruir en LAB, sin persistir en Producción, una serie propuesta para cada
cartera papel.

La reconstrucción debe usar exclusivamente:

- incorporaciones únicas;
- entrada_ts;
- sesiones NY correctas;
- anchors certificados disponibles;
- precios point-in-time;
- episodios efectivos en cada sesión;
- cierres demostrables;
- splits soportados;
- dividendos según la política simulada;
- flujos externos simulados;
- benchmark point-in-time.

No utilizar:

- composición actual aplicada hacia atrás;
- evidencia posterior para crear un episodio histórico;
- snapshots.fecha como sesión;
- precio vivo;
- SPY actual;
- holdings todavía no incorporados;
- episodios después de su salida.

Generar por cartera y por cada metodología simulada:

- primera sesión;
- última sesión;
- episodios activos por sesión;
- paper market value;
- paper cash;
- external flow;
- units outstanding;
- unit value;
- daily return;
- cumulative return;
- drawdown;
- benchmark unit value;
- benchmark return;
- cobertura;
- advertencias.

No escribir la serie simulada en las tablas de Producción.

==================================================
13. PROPUESTA DE MODELO v27
==================================================

Diseñar, pero no aplicar todavía en Producción, una migración aditiva v27.

Proponer DDL para:

A. paper_position_episode

Campos mínimos:

- episode_id;
- paper_portfolio_id;
- source_holding_id;
- ticker;
- quantity;
- incorporated_at_utc;
- market_timezone;
- incorporation_market_state;
- ny_local_datetime;
- anchor_session;
- anchor_price;
- anchor_price_type;
- anchor_price_source;
- anchor_available_at;
- episode_effective_at;
- price_basis;
- original_recorded_entry_price;
- original_recorded_notional;
- certified_paper_flow_notional;
- catalyst_snapshot_id;
- thesis_id;
- status;
- closed_at_utc;
- exit_session;
- exit_price;
- exit_reason;
- certification_state;
- resolution_reason;
- content_hash;
- created_at.

B. paper_portfolio_flow

Campos mínimos:

- flow_id;
- paper_portfolio_id;
- episode_id;
- effective_session;
- amount;
- flow_type;
- pre_flow_NAV;
- pre_flow_unit_value;
- units_delta;
- post_flow_NAV;
- post_flow_unit_value;
- source;
- external_reference;
- content_hash;
- created_at.

Tipos propuestos:

- PAPER_INITIAL_CONTRIBUTION;
- PAPER_POSITION_INCORPORATION;
- PAPER_EXTERNAL_WITHDRAWAL;
- PAPER_DIVIDEND;
- PAPER_COMMISSION;
- PAPER_CORRECTION.

No crear PAPER_POSITION_EXIT_WITHDRAWAL hasta que Omar decida que una salida
es realmente un flujo externo.

C. paper_track_snapshot

Campos mínimos:

- paper_portfolio_id;
- valuation_session;
- paper_NAV;
- paper_unit_value;
- daily_return;
- cumulative_return;
- paper_market_value;
- paper_cash;
- units_outstanding;
- active_episodes;
- external_flow;
- benchmark_unit_value;
- benchmark_return;
- drawdown;
- price_coverage_status;
- methodology;
- certification_state;
- content_hash;
- calculated_at.

Unicidad:

paper_portfolio_id + valuation_session + methodology_version

La propuesta debe incluir:

- constraints;
- índices;
- idempotencia;
- relaciones;
- estados de certificación;
- estrategia de rollback;
- estrategia de supersesión;
- preservación del historial;
- prevención de duplicados.

No aplicar la migración.

No crear todavía tablas de lifecycle de catalizadores.

Eso corresponderá a una versión posterior y a otro gate.

==================================================
14. PREVIEW DE BACKFILL
==================================================

Generar un preview de backfill, no el backfill real.

Para cada incorporación propuesta:

- identidad;
- lineage;
- anchor;
- cobertura;
- notional original;
- notional certificado;
- política de flujo simulada;
- tratamiento de salida;
- tratamiento de dividendos;
- benchmark;
- warnings;
- elegibilidad automática.

Clasificar:

AUTO_BACKFILL_SAFE

Solo si:

- lineage determinista;
- anchor disponible;
- precio coherente;
- quantity fiable;
- notional resoluble;
- sesión correcta;
- salida resoluble, si existe;
- sin conflicto de corporate actions.

MANUAL_REVIEW_REQUIRED

Cuando exista una decisión económica o de lineage pendiente.

PRICE_COVERAGE_INCOMPLETE

Cuando falte un cierre necesario.

LINEAGE_AMBIGUOUS

Cuando no pueda determinarse la incorporación única.

METHODOLOGY_DECISION_REQUIRED

Cuando el resultado dependa de una política todavía no aprobada.

No considerar AUTO_BACKFILL_SAFE un permiso para escribir.

Todo backfill queda pendiente de aprobación posterior.

==================================================
15. PRUEBAS EN LAB
==================================================

Crear o ejecutar en aislamiento:

verify-lens-paper-track-design.mjs
verify-paper-track-unitization-lab.mjs
verify-paper-track-benchmark-lab.mjs
verify-paper-track-anchor-preview.mjs

PRUEBAS TEMPORALES

A. REGULAR_OPEN

2026-07-10 23:00 Asia/Dubai
=
2026-07-10 15:00 America/New_York

- anchor_session = 2026-07-10;
- anchor disponible tras el cierre;
- episode_effective_at = cierre;
- retorno inicial = 0;
- primera variación posterior.

B. CAMBIO DE FECHA

2026-07-11 01:00 Asia/Dubai
=
2026-07-10 17:00 America/New_York

- market_state = AFTER_CLOSE;
- anchor_session = 2026-07-10.

C. PRE_MARKET

- última sesión cerrada.

D. FIN DE SEMANA

- convertir primero a Nueva York;
- resolver después el calendario.

E. FESTIVO

- última sesión bursátil cerrada.

F. DST

- fecha con UTC-04;
- fecha con UTC-05;
- sin offsets fijos.

G. SNAPSHOTS.FECHA

- demostrar que no se utiliza como sesión económica.

H. NO RETORNO PREVIO

- ningún punto anterior a episode_effective_at.

I. PRECIO AUSENTE

- no fallback;
- PRICE_COVERAGE_INCOMPLETE.

PRUEBAS DE IDENTIDAD

J. SNAPSHOT REPETIDO

- una incorporación;
- un episodio propuesto.

K. REINCORPORACIÓN

- dos incorporaciones demostrables;
- dos episodios.

L. LINEAGE AMBIGUOUS

- no backfill automático.

PRUEBAS ECONÓMICAS

M. APORTACIÓN NEUTRAL

- unit value no cambia por el flujo.

N. SALIDA KEEP_AS_PAPER_CASH

- venta interna;
- aumenta cash;
- no cambia unidades por la venta;
- no se registra retirada externa.

O. SALIDA EXTERNAL_WITHDRAWAL

- disminuyen NAV y unidades proporcionalmente;
- unit value no cambia por el flujo.

P. DIVIDENDO

- si se soporta, aumenta NAV y unit value;
- no emite unidades.

Q. COMISIÓN

- reduce NAV y unit value;
- no es flujo externo.

R. SPLIT

- cambia cantidad;
- no cambia valor económico;
- no genera retorno.

PRUEBAS DE BENCHMARK

S. DOS APORTACIONES

- dos sesiones;
- dos precios SPY;
- ninguna usa SPY actual.

T. RETIRADA EXTERNA

- misma sesión e importe que la cartera papel;
- neutral al retorno benchmark.

U. SALIDA INTERNA

- no tratarla como retirada externa sin política aprobada.

Requisitos por suite:

PASS > 0
FAIL = 0
NO_RESULT = 0

Las pruebas no deben escribir en Producción.

==================================================
16. ARTEFACTOS DE ENTREGA
==================================================

Entregar artefactos locales sin secretos:

1. preview de identidad de incorporaciones;
2. preview de anchors;
3. comparación de notionals;
4. simulación de políticas de salida;
5. simulación de dividendos/comisiones;
6. simulación de benchmark;
7. curvas LAB por cartera;
8. propuesta DDL v27;
9. preview de backfill;
10. lista de decisiones requeridas.

Los artefactos no deben incluir:

- secretos;
- contenido de .env;
- tokens;
- datos personales innecesarios;
- bases completas;
- dumps de Producción.

No añadirlos al índice sin autorización.

==================================================
17. ENTREGA OBLIGATORIA
==================================================

A. BASELINE

CANONICAL_HEAD = <hash>
SCHEMA_PRODUCTION = <versión>
LAB_ISOLATED_FROM_PRODUCTION = PASS/FAIL
PRODUCTION_READ_ONLY = PASS/FAIL
GIT_STATUS_AFTER = <estado>
NO_REMOTE_OPERATIONS = PASS/FAIL

B. IDENTIDAD

PAPER_PORTFOLIOS = <número>
PAPER_HOLDING_ROWS = <número>
PAPER_UNIQUE_INCORPORATIONS = <número>
PAPER_REPEATED_SNAPSHOT_ROWS = <número>
PAPER_REINCORPORATIONS = <número>
PAPER_AMBIGUOUS_LINEAGE = <número>

C. ANCHORS

PROPOSED_EPISODES = <número>
ANCHORS_WITH_FULL_COVERAGE = <número>
ANCHORS_PRICE_COVERAGE_INCOMPLETE = <número>
UTC_DATE_CORRECTIONS = <número>
CROSS_DATE_CASES_RESOLVED = PASS/FAIL
UTC_TO_NEW_YORK_CORRECT = PASS/FAIL
ANCHOR_USES_OFFICIAL_CLOSE = PASS/FAIL
ORIGINAL_ENTRY_DATA_PRESERVED = PASS/FAIL

D. CORPORATE ACTIONS

EPISODES_WITH_SPLITS = <número>
EPISODES_WITH_DIVIDENDS = <número>
EPISODES_WITH_PRICE_GAPS = <número>
SPLIT_HANDLING = SUPPORTED/INCOMPLETE/DECISION_REQUIRED

E. MÉTODO

PROPOSED_RETURN_METHOD =
UNITIZED_TWR /
PRICE_RETURN_TWR /
UNDETERMINED

HISTORICAL_NOTIONAL_SOURCE =
ORIGINAL_RECORDED_COST /
CERTIFIED_ANCHOR_VALUE /
FIXED_NOTIONAL /
DECISION_REQUIRED

ENTRY_FLOW_RETURN_NEUTRAL_IN_LAB = PASS/FAIL

EXIT_CAPITAL_POLICY =
KEEP_AS_PAPER_CASH /
EXTERNAL_WITHDRAWAL /
DECISION_REQUIRED

EXIT_FLOW_RETURN_NEUTRAL_IN_LAB = PASS/FAIL

DIVIDEND_POLICY =
PAPER_CASH /
REINVEST /
UNSUPPORTED /
DECISION_REQUIRED

COMMISSION_POLICY =
REDUCE_NAV /
NOT_MODELED /
DECISION_REQUIRED

F. BENCHMARK

SPY_BENCHMARK_FEASIBLE = PASS/FAIL
SPY_BENCHMARK_COVERAGE = <porcentaje>
SPY_EXTERNAL_FLOW_MATCHING = PASS/FAIL
SPY_EXIT_POLICY = <descripción/DECISION_REQUIRED>
SPY_HISTORICAL_ANCHORS_CORRECT_IN_LAB = PASS/FAIL

G. PREVIEW DE BACKFILL

AUTO_BACKFILL_SAFE = <número>
MANUAL_REVIEW_REQUIRED = <número>
PRICE_COVERAGE_INCOMPLETE = <número>
LINEAGE_AMBIGUOUS = <número>
METHODOLOGY_DECISION_REQUIRED = <número>
PREVIEW_COMPLETE = PASS/FAIL

H. PROPUESTA v27

V27_DDL_PROPOSED = PASS/FAIL
V27_ADDITIVE = PASS/FAIL
V27_IDEMPOTENT_DESIGN = PASS/FAIL
V27_ROLLBACK_DOCUMENTED = PASS/FAIL
V27_APPLIED_TO_PRODUCTION = FAIL

FAIL es el resultado correcto para V27_APPLIED_TO_PRODUCTION en este Gate 2A.

I. INTEGRIDAD

NO_PRODUCTION_SCHEMA_CHANGE = PASS/FAIL
NO_PRODUCTION_BACKFILL = PASS/FAIL
NO_PAPER_HISTORY_MUTATED = PASS/FAIL
NO_GROWTH_TRACK_CHANGED = PASS/FAIL
NO_REAL_DATA_MUTATED = PASS/FAIL
NO_THESES_CHANGED = PASS/FAIL
NO_CATALYSTS_CHANGED = PASS/FAIL
NO_HUNTING_FIELD_CHANGED = PASS/FAIL
NO_ECONOMIC_WRITES = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

==================================================
18. DECISIONES QUE DEBE PRESENTAR A OMAR
==================================================

Entregar opciones concretas y recomendación para:

1. NOTIONAL HISTÓRICO

- original recorded cost;
- certified anchor value;
- otra opción demostrable.

2. CAPITAL TRAS UNA SALIDA

- mantener como paper cash;
- retirar como flujo externo.

3. DIVIDENDOS

- aumentar paper cash;
- reinvertir;
- excluir y etiquetar PRICE_RETURN_TWR.

4. COMISIONES

- modelarlas si existen;
- declararlas no modeladas.

5. BENCHMARK

- política cuando la cartera mantiene paper cash;
- política cuando existe una retirada externa.

Para cada decisión mostrar:

- significado;
- efecto económico;
- ventajas;
- riesgos;
- efecto cuantitativo en las cinco carteras;
- recomendación de Code.

No decidir silenciosamente en nombre de Omar.

==================================================
19. SIGUIENTES SUBGATES PROPUESTOS
==================================================

Proponer, sin ejecutar:

SUBGATE 2B · MIGRACIÓN v27

- backup;
- migración estructural aditiva;
- tablas vacías;
- pruebas de migración;
- sin backfill.

SUBGATE 2C · BACKFILL

- crear episodios deterministas;
- crear flujos según decisiones aprobadas;
- preservar datos originales;
- dejar pendientes los casos incompletos.

SUBGATE 2D · MOTOR

- snapshots diarios;
- unitización;
- benchmark;
- daily-close;
- startup catch-up;
- endpoints;
- cobertura.

SUBGATE 2E · UI

- TrackRecordView compartido;
- paridad visual;
- Chrome;
- Crecimiento sin regresión.

No ejecutar ningún subgate posterior.

==================================================
20. CIERRE
==================================================

Este Gate 2A termina con:

- diseño;
- simulación;
- preview;
- propuesta;
- decisiones pendientes.

No termina con:

- migración productiva;
- backfill;
- motor productivo;
- UI;
- deploy;
- commit funcional;
- tag.

No crear todavía:

mizan-lens-paper-track-v1.0

Ese nombre queda reservado como candidato a tag final, únicamente después de
completar y validar los Subgates 2B–2E.

Detenerse y esperar aprobación expresa de Omar.