MIZAN · LA LENTE
GATE 2D-1 · CONTINUIDAD ECONÓMICA DEL TRACK PAPEL
AUDITORÍA DE TRANSACCIONES · RECONSTRUCCIÓN NAV-CONSERVING
SIN SNAPSHOTS · SIN UI · SIN ACTIVACIÓN HARD-CAP

NATURALEZA

Esta ejecución autoriza exclusivamente:

1. auditar económicamente las 249 transacciones internas existentes;
2. reconstruir NAV, cash, cantidades y unit value antes y después de cada
   composición EOD;
3. detectar transacciones nocionales que reinicien o distorsionen el capital;
4. generar una reconstrucción económica corregida en LAB;
5. proponer supersesión append-only cuando sea necesaria;
6. aplicar una migración aditiva mínima únicamente si v27 no soporta
   versionado o supersesión;
7. persistir la reconstrucción económica corregida solo tras preview,
   pruebas y backup;
8. crear un único commit local.

No autoriza:

- crear paper_track_snapshot;
- generar la serie diaria definitiva;
- modificar daily-close;
- modificar startup catch-up;
- crear endpoints productivos del Track Papel;
- modificar la UI;
- crear TrackRecordView;
- activar MANUAL_NOTIONAL_HARD_CAP;
- establecer effective_from;
- crear nuevas posiciones papel;
- crear solicitudes prospectivas;
- modificar holdings legacy;
- modificar snapshots legacy;
- modificar cat_composicion_log;
- modificar tesis;
- modificar catalizadores;
- modificar Campo de Caza;
- modificar Track de Crecimiento;
- tag;
- push;
- operaciones Git remotas.

Detenerse después de demostrar continuidad económica.

==================================================
0. BASELINE
==================================================

CANONICAL_HEAD esperado = 66059a9
schema esperado = v27
canonical root = C:/Users/support/mizan
single operational Git root = true
env-info canonical = true

Inventario estructural:

PAPER_PORTFOLIO_CONFIGS = 5
PAPER_INITIAL_FLOWS = 5
PAPER_GLOBAL_POSITIONS = 82
PAPER_MEMBERSHIP_EPISODES = 121
PAPER_OPEN_MEMBERSHIPS = 82
PAPER_CLOSED_MEMBERSHIPS = 39
PAPER_COMPOSITION_VERSIONS_EOD = 19
PAPER_COMPOSITION_MEMBERS = 249
PAPER_INTERNAL_TRANSACTIONS = 249
PAPER_TRACK_SNAPSHOTS = 0

PRICE_EVIDENCE_POINTS = 863
PRICE_MANIFEST_HASH =
3c5bf06f68db1021a491dc511470eb1a29b0c8be81bf53165ee3c1ab21cc943a

Metodología:

TRACK_METHODOLOGY =
PAPER_PRICE_RETURN_TWR_V1

HISTORICAL_TRACK_POLICY =
AS_OPERATED

HISTORICAL_ALLOCATION_POLICY =
LEGACY_EQUAL_WEIGHT_AUTO_REBALANCE

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

Política prospectiva:

MANUAL_NOTIONAL_HARD_CAP.status =
APPROVED_PENDING_ACTIVATION

MANUAL_NOTIONAL_HARD_CAP.effective_from =
NULL

No activarla.

==================================================
1. DECISIÓN SOBRE CONFIANZA HISTÓRICA
==================================================

Omar autoriza que los 36 objetos inferidos participen en la reconstrucción
económica AS_OPERATED con advertencia visible.

Distribución:

- 26 UNKNOWN_PRE_LOG;
- 10 UNKNOWN_UNLINKED.

Para ellos:

requested_notional = NULL

historical_confidence =
PARTIAL_HISTORICAL_CONFIDENCE

certification_state =
MANUAL_REVIEW_REQUIRED

No convertirlos en:

- requested_notional conocido;
- FULL_HISTORICAL_CONFIDENCE;
- evidencia exacta de la intención de Omar.

Pueden participar porque:

- el membership está resuelto;
- la composición histórica está resuelta;
- el allocated_notional legacy es reconstruible;
- excluirlos distorsionaría la cartera operada.

Declarar:

OWNER_ACCEPTED_INFERRED_OBJECTS = 36

La aceptación significa “incluir con confianza parcial”, no “certificar como
exactos”.

==================================================
2. PRECONDICIONES
==================================================

Antes de modificar:

- confirmar HEAD;
- confirmar env-info;
- confirmar schema;
- confirmar una sola raíz Git;
- inventariar archivos untracked;
- detener escrituras de aplicación;
- confirmar una sola instancia;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- validar el manifiesto de precios;
- validar contadores de Gate 2C-2;
- confirmar paper_track_snapshot = 0;
- confirmar hard cap sin effective_from.

Capturar baseline:

- las 249 transacciones;
- las 19 composiciones;
- los 249 composition members;
- configuraciones;
- flujos;
- memberships;
- holdings legacy;
- snapshots legacy;
- cat_composicion_log;
- Track de Crecimiento;
- valuations reales;
- cash real;
- NAV real;
- Book real;
- tesis;
- catalizadores.

Capturar hashes antes de cualquier escritura.

No continuar si:

- el manifiesto no coincide;
- existen precios faltantes;
- paper_track_snapshot no está vacío;
- hard cap está activo;
- integrity_check falla;
- existen violaciones de foreign keys;
- los contadores no reconcilian.

==================================================
3. DISTINCIÓN ENTRE ASIGNACIÓN LEGACY Y VALOR ECONÓMICO
==================================================

Conservar:

legacy_allocated_notional =
configured_capital / active_membership_count

Este campo representa:

- el comportamiento legacy de asignación;
- la ponderación objetivo;
- el dato necesario para reconciliar la composición antigua.

No utilizarlo después de inception como valor económico fijo de la posición.

Derivar:

legacy_target_weight_i =
legacy_allocated_notional_i
/
total_legacy_allocated_notional

Para una cartera plenamente asignada:

legacy_target_weight_i ≈ 1 / N

En cada composición económica posterior:

target_market_value_i =
legacy_target_weight_i
× NAV_PRE_TRADE

No utilizar:

target_market_value_i =
legacy_target_weight_i
× configured_capital

salvo en inception, donde:

NAV_PRE_TRADE = configured_capital

La metodología histórica queda definida como:

- memberships y pesos AS_OPERATED;
- valor económico NAV-CONSERVING;
- transacciones certificadas al cierre oficial;
- no ejecución real;
- reconstrucción papel point-in-time.

==================================================
4. ESTADO INICIAL
==================================================

Para cada cartera:

initial_contribution =
configured_capital leído de paper_portfolio_track_config.

No hardcodear 10.000 USD.

initial_unit_value =
100

initial_units =
initial_contribution / 100

initial_NAV =
initial_contribution

initial_cash_before_purchase =
initial_contribution

Para la primera composición EOD:

target_market_value_i =
initial_NAV × target_weight_i

certified_quantity_i =
target_market_value_i / official_close_i

Después de compras:

paper_cash_after =
initial_NAV - suma(target_market_value_i)

paper_NAV_after =
paper_cash_after
+ suma(certified_quantity_i × official_close_i)

Debe cumplirse:

paper_NAV_after = initial_NAV
paper_unit_value_after = 100
return_created = 0

==================================================
5. CONTINUIDAD ECONÓMICA POR COMPOSICIÓN
==================================================

Para cada cartera y transición EOD:

1. cargar cantidades económicas de la composición EOD anterior;
2. valorar esas cantidades al cierre oficial actual;
3. cargar paper cash anterior;
4. calcular NAV_PRE_TRADE;
5. cargar pesos objetivo legacy de la nueva composición;
6. calcular target market values sobre NAV_PRE_TRADE;
7. calcular target quantities al cierre actual;
8. calcular deltas netos de cantidad;
9. calcular compras y ventas netas;
10. calcular paper cash después;
11. calcular NAV_POST_TRADE;
12. comprobar unidades;
13. comprobar unit value;
14. comprobar retorno creado por la operación.

Fórmulas:

MARKET_VALUE_PRE_TRADE =
Σ(quantity_before_i × official_close_i)

NAV_PRE_TRADE =
paper_cash_before + MARKET_VALUE_PRE_TRADE

target_market_value_i =
target_weight_i × NAV_PRE_TRADE

target_quantity_i =
target_market_value_i / official_close_i

cash_delta =
-Σ((target_quantity_i - quantity_before_i) × official_close_i)

paper_cash_after =
paper_cash_before + cash_delta

MARKET_VALUE_POST_TRADE =
Σ(target_quantity_i × official_close_i)

NAV_POST_TRADE =
paper_cash_after + MARKET_VALUE_POST_TRADE

Continuidad obligatoria:

abs(NAV_POST_TRADE - NAV_PRE_TRADE) <= ECONOMIC_EPSILON

units_after =
units_before

unit_value_after =
unit_value_before

internal_trade_return =
0

==================================================
6. EPSILON Y REDONDEO
==================================================

Definir separadamente:

MONETARY_STORAGE_PRECISION
QUANTITY_STORAGE_PRECISION
ECONOMIC_EPSILON
DISPLAY_CASH_EPSILON

No utilizar una única tolerancia para todo.

ECONOMIC_EPSILON debe ser suficientemente pequeña para detectar pérdidas o
ganancias materiales, pero compatible con:

- precisión monetaria;
- precisión de quantity;
- número de posiciones;
- redondeo acumulado.

Si:

abs(paper_cash) <= DISPLAY_CASH_EPSILON

la UI futura podrá mostrar:

0,00 USD

Pero la persistencia debe seguir una regla determinista.

No ocultar:

- cash negativo material;
- diferencia de NAV;
- pérdida;
- beneficio;
- error de precio.

Entregar:

MAX_NAV_CONTINUITY_DIFFERENCE
MAX_UNIT_VALUE_CONTINUITY_DIFFERENCE
MAX_ROUNDING_RESIDUAL
MIN_ECONOMIC_PAPER_CASH

==================================================
7. REASIGNACIONES PRE-FILL
==================================================

Las 39 reasignaciones ocurrieron antes del fill.

Reglas:

CARTERA DE ORIGEN

- membership de auditoría cerrado;
- cero economic quantity;
- cero market value;
- cero venta;
- cero proceeds;
- cero turnover;
- cero impacto en NAV;
- cero impacto en units.

CARTERA DE DESTINO

- membership económico inicia en la sesión del fill;
- compra económica al cierre oficial;
- entra en la composición EOD;
- target market value calculado sobre NAV_PRE_TRADE del destino.

No generar transacciones económicas para origen cuando:

economic_quantity_before = 0

Declarar:

PREFILL_REASSIGNMENT_ORIGIN_SALES = 0
PREFILL_REASSIGNMENT_ORIGIN_TURNOVER = 0

==================================================
8. EVENTOS MÚLTIPLES EN UNA SESIÓN
==================================================

Mantener:

- cat_composicion_log como auditoría intradía;
- 19 composiciones económicas EOD.

La transición económica se calcula entre:

- última composición EOD anterior;
- composición EOD final actual.

No generar transacciones para cada cambio intradía si todos se valoran al mismo
cierre.

Netear por:

- portfolio;
- ticker;
- session;
- reconstruction version.

Declarar:

SAME_SESSION_EVENTS_NETTED = PASS/FAIL
ARTIFICIAL_TURNOVER_CREATED = 0

==================================================
9. AUDITORÍA DE LAS 249 TRANSACCIONES
==================================================

Clasificar cada transacción existente:

A. ECONOMICALLY_VALID

Cumple:

- cantidad correcta;
- precio correcto;
- cash delta correcto;
- NAV continuity;
- unit continuity;
- cero retorno interno;
- no venta pre-fill artificial.

B. LEGACY_NOTIONAL_AUDIT_ONLY

Representa correctamente:

- allocated_notional legacy;
- peso;
- composición;

pero no debe utilizarse como transacción económica del motor.

C. REQUIRES_SUPERSESSION

Generaría:

- reinicio a configured capital;
- destrucción de beneficio o pérdida;
- cash incorrecto;
- quantity incorrecta;
- turnover artificial;
- discontinuidad NAV/unit value.

Entregar:

EXISTING_TRANSACTIONS_ECONOMICALLY_VALID
EXISTING_TRANSACTIONS_LEGACY_AUDIT_ONLY
EXISTING_TRANSACTIONS_REQUIRES_SUPERSESSION

No modificar ni borrar destructivamente las transacciones existentes.

==================================================
10. RECONSTRUCCIÓN ECONÓMICA VERSIONADA
==================================================

Si alguna transacción no es económicamente válida:

Crear una nueva:

ECONOMIC_RECONSTRUCTION_VERSION

La versión debe contener o permitir derivar:

- portfolio;
- session;
- previous composition;
- target composition;
- NAV_PRE_TRADE;
- target weights;
- target market values;
- quantities before;
- quantities after;
- cash before;
- cash after;
- NAV_POST_TRADE;
- transaction deltas;
- source composition;
- price evidence;
- confidence;
- superseded transaction reference;
- methodology;
- content hash.

Las transacciones anteriores:

- se conservan;
- no se borran;
- no se editan en-place;
- se marcan como no canónicas para el Track;
- siguen disponibles como auditoría legacy.

Las nuevas transacciones:

- son append-only;
- tienen claves de idempotencia;
- referencian la versión económica;
- son las únicas utilizadas posteriormente por Gate 2D-2.

==================================================
11. SCHEMA
==================================================

Primero determinar si v27 soporta de forma segura:

- reconstruction_version;
- canonical_for_track;
- supersedes_transaction_id;
- NAV before/after;
- cash before/after;
- quantity before/after;
- confidence;
- content hash.

Si v27 ya lo soporta:

- no migrar.

Si no lo soporta:

- proponer migración aditiva mínima;
- utilizar NEXT_AVAILABLE_SCHEMA_VERSION;
- no asumir automáticamente v28;
- revisar el registro y reservas de versiones;
- no incluir lifecycle de catalizadores;
- no modificar tablas reales;
- no reutilizar campos con otra semántica.

Antes de aplicar una migración:

- entregar DDL;
- probar en LAB;
- probar idempotencia;
- probar rollback vacío;
- backup;
- integrity;
- foreign keys.

Si la migración excede el soporte mínimo de supersesión:

- detenerse;
- no ampliar el alcance.

==================================================
12. PREVIEW ANTES DE ESCRIBIR
==================================================

Generar un preview por cartera y composición.

Campos:

- session;
- NAV_PRE_TRADE;
- paper cash before;
- market value before;
- target weights;
- target market values;
- quantities before;
- quantities after;
- net trades;
- paper cash after;
- market value after;
- NAV_POST_TRADE;
- NAV difference;
- units before;
- units after;
- unit value before;
- unit value after;
- return created;
- confidence;
- transaction classification;
- supersession required.

Resumen por cartera:

- initial NAV;
- final reconstructed NAV;
- maximum NAV discontinuity;
- maximum unit discontinuity;
- transaction count;
- turnover;
- cash residual;
- confidence;
- warnings.

No escribir si:

- existe NAV discontinuity fuera de epsilon;
- existe unit discontinuity;
- existe retorno creado por transacción;
- existe cash negativo material;
- existe venta pre-fill en origen;
- falta precio;
- falta composición;
- existe lineage sin resolver.

==================================================
13. PERSISTENCIA
==================================================

Solo después de preview verde.

Antes:

- detener escrituras;
- una sola instancia;
- backup;
- SHA-256;
- abrir backup;
- integrity_check;
- foreign_key_check;
- baseline completo.

Persistir:

- reconstruction version;
- transacciones económicas corregidas;
- relaciones de supersesión;
- estado canónico/no canónico;
- referencias de evidencia;
- hashes.

No persistir:

- paper_track_snapshot;
- benchmark snapshots;
- endpoints;
- cambios UI;
- effective_from hard cap.

La operación debe ser transaccional.

Si falla:

- rollback completo;
- reconstrucción anterior de Gate 2C-2 permanece intacta;
- paper_track_snapshot sigue vacío.

La segunda ejecución:

- devuelve ALREADY_CURRENT;
- no duplica;
- no cambia hashes;
- no crea snapshots.

==================================================
14. PRUEBAS
==================================================

Crear:

verify-paper-economic-continuity.mjs
verify-paper-nav-preservation.mjs
verify-paper-unit-continuity.mjs
verify-paper-internal-trade-neutrality.mjs
verify-paper-prefill-reassignment.mjs
verify-paper-same-session-netting.mjs
verify-paper-economic-reconstruction-version.mjs
verify-paper-economic-supersession.mjs
verify-paper-economic-idempotency.mjs
verify-paper-economic-isolation.mjs

Casos:

A. Ganancia antes del rebalanceo.

- initial NAV = 10.000;
- market gain = 10 %;
- NAV_PRE_TRADE = 11.000;
- nueva composición se pondera sobre 11.000;
- NAV_POST_TRADE = 11.000;
- unit value conserva +10 %.

B. Pérdida antes del rebalanceo.

- NAV_PRE_TRADE = 9.000;
- nueva composición se pondera sobre 9.000;
- no reiniciar a 10.000;
- pérdida permanece.

C. Rebalanceo sin mercado.

- NAV no cambia;
- unit value no cambia;
- retorno interno = 0.

D. Nueva membership.

- target weights sobre NAV actual;
- no sobre configured capital.

E. Pre-fill reassignment.

- cero venta y turnover en origen.

F. Eventos misma sesión.

- una transición EOD neta;
- cero turnover artificial.

G. Cash residual.

- redondeo dentro de tolerancia;
- diferencias materiales bloqueadas.

H. Supersesión.

- legacy preservado;
- versión económica nueva;
- solo versión canónica utilizada.

I. Idempotencia.

- segunda ejecución sin cambios.

J. Aislamiento.

- Track real, cash real y NAV real intactos.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

==================================================
15. VERIFICACIÓN DE CRECIMIENTO Y DATOS REALES
==================================================

Antes y después comparar:

- payload de /track/crecimiento;
- serie;
- métricas;
- ventanas;
- benchmark;
- drawdown;
- valuations;
- holdings;
- movimientos;
- cash;
- NAV;
- Book.

Resultado:

GROWTH_TRACK_UNCHANGED = PASS
REAL_BOOK_UNCHANGED = PASS
REAL_CASH_UNCHANGED = PASS
REAL_NAV_UNCHANGED = PASS
REAL_VALUATIONS_UNCHANGED = PASS

Si existe un campo temporal no determinista:

- normalizarlo;
- comparar contenido económico;
- documentar la diferencia.

==================================================
16. CÓDIGO Y COMMIT
==================================================

Añadir únicamente:

- auditor económico;
- reconstrucción NAV-conserving;
- soporte mínimo de versionado/supersesión;
- pruebas;
- documentación.

No añadir:

- snapshots diarios;
- endpoints;
- UI;
- hard-cap funcional;
- scratchpad;
- backups;
- bases;
- datos productivos;
- secretos.

Ejecutar secret scan.

Crear un commit local:

fix(lens): preserve paper NAV across historical rebalances

No crear tag.
No hacer push.

==================================================
17. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE = 66059a9
CANONICAL_HEAD_AFTER = <hash>

SCHEMA_BEFORE = v27
SCHEMA_AFTER = <v27/NEXT_AVAILABLE_SCHEMA_VERSION>

OWNER_ACCEPTED_INFERRED_OBJECTS = 36
PARTIAL_HISTORICAL_CONFIDENCE_PRESERVED = PASS/FAIL

COMPOSITIONS_AUDITED = <número>/19
TRANSACTIONS_AUDITED = <número>/249

EXISTING_TRANSACTIONS_ECONOMICALLY_VALID = <número>
EXISTING_TRANSACTIONS_LEGACY_AUDIT_ONLY = <número>
EXISTING_TRANSACTIONS_REQUIRES_SUPERSESSION = <número>

ECONOMIC_RECONSTRUCTION_VERSION = <versión>
TRANSACTIONS_SUPERSEDED = <número>
CANONICAL_ECONOMIC_TRANSACTIONS_CREATED = <número>

INTERNAL_TRANSACTION_NAV_CONTINUITY = PASS/FAIL
INTERNAL_TRANSACTION_UNIT_CONTINUITY = PASS/FAIL
INTERNAL_TRANSACTION_RETURN_CREATED = <valor>

MAX_NAV_CONTINUITY_DIFFERENCE = <valor>
MAX_UNIT_VALUE_CONTINUITY_DIFFERENCE = <valor>
MAX_ROUNDING_RESIDUAL = <valor>
MIN_ECONOMIC_PAPER_CASH = <tabla>

PREFILL_REASSIGNMENT_ORIGIN_SALES = <número>
PREFILL_REASSIGNMENT_ORIGIN_TURNOVER = <valor>
SAME_SESSION_EVENTS_NETTED = PASS/FAIL
ARTIFICIAL_TURNOVER_CREATED = <valor>

PAPER_TRACK_SNAPSHOTS_CREATED = 0

PROSPECTIVE_POLICY_STATUS =
APPROVED_PENDING_ACTIVATION

PROSPECTIVE_POLICY_EFFECTIVE_FROM =
NULL

NO_POLICY_ACTIVATED = PASS/FAIL
GROWTH_TRACK_UNCHANGED = PASS/FAIL
REAL_BOOK_UNCHANGED = PASS/FAIL
REAL_CASH_UNCHANGED = PASS/FAIL
REAL_NAV_UNCHANGED = PASS/FAIL
REAL_VALUATIONS_UNCHANGED = PASS/FAIL
HOLDINGS_LEGACY_UNCHANGED = PASS/FAIL
SNAPSHOTS_LEGACY_UNCHANGED = PASS/FAIL
THESES_UNCHANGED = PASS/FAIL
CATALYSTS_UNCHANGED = PASS/FAIL
HUNTING_FIELD_UNCHANGED = PASS/FAIL
NO_REAL_ECONOMIC_WRITES = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

LOCAL_COMMIT = <hash>

EXACT_BLOCKERS_BEFORE_GATE_2D2:
1. <bloqueo>
2. <bloqueo>
3. <bloqueo>

Detenerse después del Gate 2D-1.