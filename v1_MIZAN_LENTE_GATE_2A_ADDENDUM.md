MIZAN · LA LENTE
GATE 2A · ADDENDUM OBLIGATORIO
TOPOLOGÍA DEL CAPITAL PAPEL Y DISTINCIÓN ENTRE FLUJOS EXTERNOS Y OPERACIONES INTERNAS

CONTEXTO

El Gate 2A queda aceptado en:

- identidad de 82 incorporaciones únicas;
- entrada_ts UTC exacto;
- conversión a America/New_York;
- 28 correcciones de sesión;
- propuesta de anchors EOD;
- viabilidad técnica de una serie unitizada;
- separación completa respecto al patrimonio real.

No queda aprobado todavía el Subgate 2B.

Existe una cuestión metodológica pendiente:

Las cinco carteras parecen haberse construido con un capital fijo de 10.000
y posiciones equal-weight.

Por tanto, no debe asumirse que cada incorporación de una posición representa
una nueva aportación externa.

MISIÓN

Determinar en LAB, sin cambiar Producción, la topología real del capital de
cada cartera papel.

Distinguir estrictamente:

A. FLUJO EXTERNO

Cambia el capital aportado y las unidades:

- aportación inicial;
- aportación posterior;
- retirada externa.

B. OPERACIÓN INTERNA

No cambia las unidades:

- compra de posición;
- venta de posición;
- traslado entre paper_cash y paper_market_value;
- dividendo;
- comisión;
- rebalanceo.

No ejecutar migración.
No realizar backfill.
No modificar UI.
No desplegar.
No hacer commit, tag ni push.

==================================================
1. CAPITAL INICIAL POR CARTERA
==================================================

Para cada una de las cinco carteras determinar:

- portfolio_id;
- nombre;
- fecha de creación;
- primera incorporación;
- capital nominal configurado;
- importe total registrado;
- número de posiciones iniciales;
- importe registrado por posición;
- suma de importes;
- cantidades originales;
- precios originales;
- sesiones de anchor;
- dispersión temporal de las incorporaciones;
- cash_pct configurado;
- evidencia de equal-weight.

Comprobar específicamente si:

sum(importe_registrado_i) ≈ 10.000

para cada cartera.

No limitarse a comprobar que todos los importes son iguales.

Declarar:

PAPER_INITIAL_CAPITAL_BY_PORTFOLIO = <tabla>
PAPER_RECORDED_NOTIONAL_SUM_BY_PORTFOLIO = <tabla>
PAPER_EQUAL_WEIGHT_CONFIRMED = PASS/FAIL
PAPER_FIXED_CAPITAL_10000_CONFIRMED = PASS/FAIL

==================================================
2. AGRUPACIÓN DE INCORPORACIONES
==================================================

Agrupar las 82 incorporaciones por:

- cartera;
- sesión de anchor;
- fecha/hora de incorporación;
- proceso o tesis que las creó.

Determinar si cada cartera representa:

A. INITIAL_BATCH

Todas o la mayoría de las posiciones forman una cartera inicial financiada
por una única aportación de 10.000.

B. STAGGERED_INITIAL_DEPLOYMENT

Existe una aportación inicial de 10.000 y las posiciones se incorporan en
sesiones distintas utilizando paper_cash disponible.

C. REPEATED_EXTERNAL_CONTRIBUTIONS

Cada incorporación recibió realmente capital externo adicional.

D. UNDETERMINED

No existe evidencia suficiente.

No elegir C únicamente porque cada posición tenga un importe registrado.

El importe puede representar una asignación interna del capital inicial.

Entregar por cartera:

- primera anchor_session;
- última anchor_session inicial;
- número de sesiones de despliegue;
- importe desplegado por sesión;
- cash restante después de cada sesión;
- evidencia del modelo.

==================================================
3. MODELO CANÓNICO A SIMULAR
==================================================

Simular como opción principal:

PAPER_INITIAL_CAPITAL =
10.000 por cartera

En la primera sesión efectiva:

- registrar una aportación externa inicial;
- emitir unidades a paper_unit_value = 100;
- aumentar paper_cash en 10.000.

Cada incorporación:

- no es un flujo externo;
- no emite unidades;
- reduce paper_cash;
- aumenta paper_market_value;
- compra una cantidad sintética al anchor oficial;
- empieza con P&L 0.

Identidad:

paper_NAV =
paper_cash + paper_market_value

Compra interna:

paper_cash_after =
paper_cash_before - acquisition_notional

paper_market_value_after =
paper_market_value_before + acquisition_notional

units_outstanding_after =
units_outstanding_before

unit_value_after =
unit_value_before

La compra no debe crear ni destruir retorno.

Si las incorporaciones ocurren en varias sesiones:

- el capital todavía no desplegado permanece como paper_cash;
- la cartera obtiene retorno solo sobre el capital invertido;
- el efectivo no genera retorno salvo que exista una metodología explícita
  para intereses.

No permitir paper_cash negativo.

==================================================
4. NOTIONAL Y CANTIDAD CERTIFICADA
==================================================

Evaluar esta propuesta:

PLANNED_EQUAL_WEIGHT_NOTIONAL =
importe registrado de la posición

Si la evidencia confirma capital 10.000 equal-weight:

planned_notional_i =
10.000 / número objetivo de posiciones

Para el Track certificado:

certified_quantity_i =
planned_notional_i / official_anchor_close_i

Conservar separadamente:

- original_quantity;
- original_recorded_entry_price;
- original_recorded_notional;
- planned_equal_weight_notional;
- certified_anchor_price;
- certified_quantity.

No sobrescribir cantidades ni precios originales.

No denominar este método simplemente CERTIFIED_ANCHOR_VALUE.

Clasificación propuesta:

HISTORICAL_NOTIONAL_SOURCE =
PLANNED_EQUAL_WEIGHT_NOTIONAL

QUANTITY_METHOD =
SYNTHETIC_QUANTITY_AT_CERTIFIED_ANCHOR

La UI y metodología deben indicar que:

- se trata de una reconstrucción certificada de cartera papel;
- la cantidad certificada puede diferir de la cantidad originalmente
  registrada;
- el historial original permanece disponible para auditoría.

Comparar frente a:

A. ORIGINAL_RECORDED_COST
B. ORIGINAL_QUANTITY_AT_CERTIFIED_ANCHOR
C. PLANNED_EQUAL_WEIGHT_NOTIONAL

Mostrar el efecto de las tres alternativas en:

- capital inicial;
- cash;
- NAV;
- retorno;
- drawdown;
- pesos;
- benchmark.

==================================================
5. VIABILIDAD DEL CASH PAPEL
==================================================

Reconstruir sesión por sesión usando:

- aportación inicial de 10.000;
- planned_notional;
- certified quantity;
- anchors propuestos.

Verificar para cada cartera:

- paper_cash inicial;
- compras por sesión;
- paper_cash posterior;
- mínimo paper_cash;
- si alguna compra genera cash negativo;
- saldo final tras despliegue;
- diferencias por redondeo.

Si existe cash negativo:

- no aumentar capital automáticamente;
- identificar la causa;
- comprobar redondeos;
- comprobar si el número objetivo de posiciones cambió;
- comprobar si existieron aportaciones posteriores;
- marcar CAPITAL_MODEL_INCONSISTENT.

Permitir una tolerancia de redondeo documentada, no una aportación implícita.

Entregar:

PAPER_CASH_MIN_BY_PORTFOLIO = <tabla>
PAPER_CASH_FINAL_BY_PORTFOLIO = <tabla>
NEGATIVE_PAPER_CASH_CASES = <número>
CAPITAL_MODEL_CONSISTENT = PASS/FAIL

==================================================
6. POLÍTICA DE SALIDA
==================================================

Simular y recomendar:

EXIT_CAPITAL_POLICY =
KEEP_AS_PAPER_CASH

Al cerrar una posición:

1. valorar hasta el precio de salida certificado;
2. vender la posición;
3. reducir paper_market_value;
4. aumentar paper_cash;
5. mantener units_outstanding;
6. no registrar flujo externo;
7. no modificar unit_value por la compraventa en sí;
8. conservar el P&L económico acumulado.

Una salida no debe crear:

- PAPER_EXTERNAL_WITHDRAWAL;
- units_redeemed;
- una aportación negativa;

salvo que exista una decisión explícita de retirar capital de la cartera.

Dado que actualmente existen 0 salidas históricas:

- declarar la política prospectiva;
- no inventar eventos pasados.

==================================================
7. DIVIDENDOS Y COMISIONES
==================================================

Adoptar para la primera versión:

PROPOSED_RETURN_METHOD =
PRICE_RETURN_TWR

DIVIDEND_POLICY =
UNSUPPORTED

Mostrar:

INCOME_NOT_INCLUDED

No denominar la serie total-return.

Diseñar compatibilidad futura para:

dividendo fiable
→ paper_cash
→ aumenta NAV y unit_value
→ no emite unidades
→ no es flujo externo.

COMMISSION_POLICY =
NOT_MODELED

No inventar comisiones.

Si en el futuro se registran:

- reducen paper_cash;
- reducen NAV y unit_value;
- no son flujo externo.

==================================================
8. BENCHMARK SPY
==================================================

Simular la política recomendada:

El benchmark replica únicamente flujos externos.

Para cada cartera:

- aportación externa inicial de 10.000;
- inversión virtual en SPY al cierre oficial de la sesión inicial;
- emisión de unidades benchmark;
- ninguna compra interna de la cartera genera flujo benchmark;
- ninguna venta interna genera retirada benchmark.

Si la cartera vende una posición y conserva paper_cash:

- el benchmark permanece invertido en SPY;
- no crear benchmark cash;
- no vender SPY;
- la diferencia refleja la decisión activa de mantener efectivo.

Solo una verdadera retirada externa debe retirar unidades benchmark.

Declarar:

SPY_BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

Explicar que esta política compara:

- estrategia papel activa, incluida su asignación a cash;
- inversión pasiva en SPY con el mismo capital externo.

Simular también la alternativa CASH_MATCHED únicamente para comparación,
pero no recomendarla salvo que el objetivo del benchmark sea neutralizar la
decisión de asignación a efectivo.

==================================================
9. LEDGER PAPEL PROPUESTO
==================================================

Corregir la propuesta v27 para distinguir:

FLUJOS EXTERNOS QUE CAMBIAN UNIDADES:

- PAPER_INITIAL_CONTRIBUTION;
- PAPER_EXTERNAL_CONTRIBUTION;
- PAPER_EXTERNAL_WITHDRAWAL;
- PAPER_CORRECTION, solo con clasificación explícita.

EVENTOS ECONÓMICOS INTERNOS QUE NO CAMBIAN UNIDADES:

- PAPER_POSITION_PURCHASE;
- PAPER_POSITION_SALE;
- PAPER_DIVIDEND;
- PAPER_COMMISSION;
- PAPER_SPLIT_ADJUSTMENT.

No modelar PAPER_POSITION_INCORPORATION como aportación externa por defecto.

Puede utilizarse:

- una tabla general paper_portfolio_event con event_class; o
- tablas separadas para external flows e internal events.

Si se conserva paper_portfolio_flow:

- debe contener únicamente flujos externos;
- compras, ventas, dividendos y comisiones deben quedar en una entidad de
  eventos internos o en el lifecycle del episodio.

Proponer el DDL corregido.

No aplicarlo.

==================================================
10. PRUEBAS LAB OBLIGATORIAS
==================================================

A. CAPITAL INICIAL

- aportación externa 10.000;
- unit_value = 100;
- units = 100;
- paper_cash = 10.000.

B. COMPRA INTERNA

- compra 588,24;
- cash disminuye;
- market value aumenta;
- NAV no cambia;
- unidades no cambian;
- unit value no cambia.

C. DESPLIEGUE ESCALONADO

- varias compras en sesiones distintas;
- cash permanece hasta cada compra;
- no se emiten unidades adicionales;
- el cash no invertido no obtiene retorno.

D. INCORPORACIÓN NO ES APORTACIÓN

- alta de una posición;
- cero external flow;
- cero units issued;
- cero retorno artificial.

E. CASH NEGATIVO

- bloquear compra interna cuando supera paper_cash;
- no crear aportación automática.

F. SALIDA A CASH

- venta aumenta paper_cash;
- unidades no cambian;
- compraventa neutral en el instante;
- el P&L previo permanece en unit value.

G. BENCHMARK

- SPY recibe 10.000 iniciales;
- compras internas no afectan unidades SPY;
- ventas internas no afectan unidades SPY;
- solo flujos externos cambian unidades benchmark.

H. NOTIONAL

Comparar:

- coste original;
- original quantity × anchor;
- planned equal-weight notional.

I. AISLAMIENTO

- cero cambios en Producción;
- cero cambios económicos;
- cero schema changes.

Requisitos:

PASS > 0
FAIL = 0
NO_RESULT = 0

==================================================
11. ENTREGA
==================================================

PAPER_FIXED_CAPITAL_10000_CONFIRMED = PASS/FAIL
PAPER_EQUAL_WEIGHT_CONFIRMED = PASS/FAIL

Por cartera:

- portfolio_id;
- number_of_positions;
- recorded_notional_sum;
- first_anchor_session;
- last_initial_anchor_session;
- deployment_sessions;
- minimum_paper_cash;
- final_paper_cash;
- capital_model.

CAPITAL_TOPOLOGY =
SINGLE_INITIAL_CONTRIBUTION /
STAGGERED_INITIAL_DEPLOYMENT /
REPEATED_EXTERNAL_CONTRIBUTIONS /
MIXED /
UNDETERMINED

PROPOSED_EXTERNAL_FLOW_MODEL =
ONE_INITIAL_10000_PER_PORTFOLIO /
OTHER /
DECISION_REQUIRED

HISTORICAL_NOTIONAL_SOURCE =
PLANNED_EQUAL_WEIGHT_NOTIONAL /
ORIGINAL_RECORDED_COST /
ORIGINAL_QUANTITY_AT_CERTIFIED_ANCHOR /
DECISION_REQUIRED

QUANTITY_METHOD =
SYNTHETIC_QUANTITY_AT_CERTIFIED_ANCHOR /
ORIGINAL_QUANTITY /
DECISION_REQUIRED

NEGATIVE_PAPER_CASH_CASES = <número>
CAPITAL_MODEL_CONSISTENT = PASS/FAIL

PROPOSED_RETURN_METHOD = PRICE_RETURN_TWR
EXIT_CAPITAL_POLICY = KEEP_AS_PAPER_CASH
DIVIDEND_POLICY = UNSUPPORTED
COMMISSION_POLICY = NOT_MODELED

SPY_BENCHMARK_POLICY =
EXTERNAL_FLOWS_ONLY_FULLY_INVESTED

PAPER_FLOW_DDL_CORRECTED = PASS/FAIL
INTERNAL_EVENTS_SEPARATED_FROM_EXTERNAL_FLOWS = PASS/FAIL

NO_PRODUCTION_SCHEMA_CHANGE = PASS/FAIL
NO_BACKFILL_EXECUTED = PASS/FAIL
NO_PAPER_HISTORY_MUTATED = PASS/FAIL
NO_ECONOMIC_WRITES = PASS/FAIL
NO_PUSH_PERFORMED = PASS/FAIL

EXACT_REMAINING_DECISIONS_FOR_OMAR:
1. <decisión real pendiente, si existe>
2. <decisión real pendiente, si existe>

Detenerse.