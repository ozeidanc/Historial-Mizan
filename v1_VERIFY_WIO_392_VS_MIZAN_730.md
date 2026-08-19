RECONCILIACIÓN DE RENTABILIDAD · CIERRE 2026-08-06 · READ-ONLY

Las capturas de Wio permiten reproducir exactamente su métrica:

WIO_CURRENT_MARKET_VALUE =
4496.64 USD

WIO_CURRENT_UNREALIZED_PNL =
169.53 USD

WIO_CURRENT_OPEN_COST =
4496.64 - 169.53 =
4327.11 USD

WIO_RETURN =
169.53 / 4327.11 × 100 =
3.917857%

WIO_UI_DISPLAY =
3.92%

Además, la suma de los P&L individuales mostrados en las capturas es
exactamente 169.53 USD.

Conclusión documental inicial:

WIO_METRIC =
CURRENT_OPEN_POSITIONS_UNREALIZED_RETURN

La etiqueta “All time” se refiere al resultado desde la adquisición de las
posiciones actualmente abiertas; no incluye el realizado histórico de
posiciones cerradas.

No modificar código.
No modificar movimientos.
No revaluar.
No invalidar cachés.
No cambiar la UI.

==================================================
1. EXTRAER LA VALORACIÓN MIZAN DEL 2026-08-06
==================================================

Entregar desde la valoración persistida activa del cierre:

valuation_id
economic_date
created_at
price_as_of
movement_cutoff
formula_version
realized_pnl
unrealized_pnl
live_open_cost
market_value
cash
position_count
total_pnl
return_pct

No utilizar el estado live anterior al cierre.

Reproducir:

MIZAN_DISPLAYED_RETURN =
7.30% aproximadamente, con el valor exacto persistido

MIZAN_TOTAL_RETURN =
(realized_pnl + unrealized_pnl) / live_open_cost × 100

MIZAN_OPEN_POSITION_RETURN =
unrealized_pnl / live_open_cost × 100

MIZAN_REALIZED_COMPONENT_PP =
realized_pnl / live_open_cost × 100

Exigir:

MIZAN_TOTAL_RETURN =
MIZAN_OPEN_POSITION_RETURN
+ MIZAN_REALIZED_COMPONENT_PP

==================================================
2. COMPARAR MÉTRICAS EQUIVALENTES
==================================================

Comparar únicamente:

WIO_CURRENT_UNREALIZED_PNL =
169.53

WIO_CURRENT_OPEN_COST =
4327.11

WIO_CURRENT_POSITION_RETURN =
3.917857%

contra:

MIZAN_CURRENT_UNREALIZED_PNL =
<valor>

MIZAN_CURRENT_OPEN_COST =
<valor>

MIZAN_CURRENT_POSITION_RETURN =
<valor>

Entregar:

OPEN_PNL_DIFFERENCE_USD =
Mizan unrealized - 169.53

OPEN_COST_DIFFERENCE_USD =
Mizan open cost - 4327.11

OPEN_RETURN_DIFFERENCE_PP =
Mizan open-position return - 3.917857%

No comparar directamente el total de Mizan con el porcentaje de posiciones
abiertas de Wio.

==================================================
3. DESCOMPONER 7.30% VS 3.92%
==================================================

Calcular:

TOTAL_DISPLAY_DIFFERENCE_PP =
Mizan total return - Wio current-position return

Descomponer exactamente en:

MIZAN_REALIZED_COMPONENT_PP
OPEN_POSITION_PNL_DIFFERENCE_PP
OPEN_COST_DIFFERENCE_EFFECT_PP
ROUNDING_EFFECT_PP
UNEXPLAINED_EFFECT_PP

Exigir:

UNEXPLAINED_EFFECT_PP =
0.0000

Determinar:

DIFFERENCE_IS_ONLY_METHODOLOGY =
YES/NO

==================================================
4. RECONCILIAR POSICIONES DE LAS CAPTURAS
==================================================

Usar las cantidades visibles de Wio para el cierre 2026-08-06.

Comparar por ticker:

ticker
Wio quantity
Mizan quantity
quantity difference
Wio market value
Mizan market value
market value difference
Wio unrealized P&L
Mizan unrealized P&L
P&L difference
status

Tickers visibles:

AMD
AMZN
AAPL
APP
ARM
ASML
BKNG
CDNS
FTNT
KLAC
LRCX
MRVL
MELI
MU
MSFT
MPWR
PLTR
ROP
SNDK
STX
SHOP
TTWO
TXN
WDC

Controles:

WIO_POSITION_COUNT =
24

SUM_WIO_MARKET_VALUES =
4496.64

SUM_WIO_UNREALIZED_PNL =
169.53

Comprobar especialmente los movimientos internos del 2026-08-06:

ventas:
ADI
ADSK
ADBE
CRM
INTU
NVDA

compras:
ARM
CDNS
KLAC
MELI
SHOP

Siguen siendo:

INTERNAL_UNRECONCILED

hasta importar sus trade confirmations.

==================================================
5. DETERMINAR SI HAY ERROR EN MIZAN
==================================================

Clasificar el resultado:

A. METRICS_DIFFERENT_BUT_DATA_RECONCILED

Wio muestra únicamente el resultado de posiciones abiertas y Mizan añade
realizado histórico, pero cantidades, coste y no realizado coinciden.

B. METRICS_DIFFERENT_AND_OPEN_LEDGER_MISMATCH

Además de utilizar fórmulas distintas, Mizan tiene diferencias en cantidades,
costes, precios o P&L de posiciones abiertas.

C. MIZAN_VALUATION_ERROR

La valoración del cierre utiliza precios, movimientos o cálculos incorrectos.

D. INSUFFICIENT_DETAIL

Entregar una sola clasificación principal y la evidencia.

==================================================
6. SEMÁNTICA DE MIZAN
==================================================

Confirmar la definición exacta de la métrica visible:

MIZAN_METRIC_NAME =
<valor>

MIZAN_METRIC_FORMULA =
<valor>

Si utiliza realized histórico + unrealized actual sobre open cost:

MIZAN_METRIC_CLASSIFICATION =
TOTAL_BOOK_PNL_ON_CURRENT_OPEN_COST

COMPARABLE_TO_WIO_3_92 =
NO

No proponer todavía copiar la fórmula ni la etiqueta de Wio.

==================================================
CHECKPOINT
==================================================

WIO_MARKET_VALUE =
4496.64

WIO_UNREALIZED_PNL =
169.53

WIO_OPEN_COST =
4327.11

WIO_OPEN_POSITION_RETURN =
3.917857%

WIO_UI_DISPLAY =
3.92%

MIZAN_DISPLAYED_RETURN =
<valor exacto>

MIZAN_REALIZED_PNL =
<valor>

MIZAN_UNREALIZED_PNL =
<valor>

MIZAN_LIVE_OPEN_COST =
<valor>

MIZAN_TOTAL_RETURN =
<valor>

MIZAN_OPEN_POSITION_RETURN =
<valor>

MIZAN_REALIZED_COMPONENT_PP =
<valor>

OPEN_PNL_DIFFERENCE_USD =
<valor>

OPEN_COST_DIFFERENCE_USD =
<valor>

METRICS_COMPARABLE =
YES/NO

DIFFERENCE_CLASSIFICATION =
<A/B/C/D>

UNEXPLAINED_EFFECT_PP =
<valor>

SAFE_TO_CHANGE_UI =
YES/NO

SAFE_TO_CHANGE_FORMULA =
YES/NO

DATA_WRITES =
0

CODE_CHANGES =
0