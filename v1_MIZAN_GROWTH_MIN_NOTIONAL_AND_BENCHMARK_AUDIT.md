MIZAN · HOTFIX PREVIO A LA PRIMERA REVISIÓN C1
MÍNIMO EJECUTABLE DE 2 USD + AUDITORÍA DEL BENCHMARK

PRIORIDAD =
URGENTE

BASELINE ESPERADO

HEAD =
4d9839f

SCHEMA =
v31

C1_STATUS =
ACTIVE

C1_EFFECTIVE_FROM =
2026-07-28

FIRST_C1_REVIEW_STATUS =
WAITING_FOR_WINDOW

La primera revisión C1 está prevista después del cierre de la sesión del
28 de julio de 2026, con review_date 2026-07-29.

Objetivo:

1. impedir recomendaciones ejecutables de crecimiento inferiores a 2 USD;
2. auditar y, si corresponde, corregir la comparación cartera–S&P 500;
3. completar el hotfix antes de publicar la primera revisión C1;
4. no modificar operaciones históricas;
5. no enviar órdenes a Wio;
6. no modificar Gate 2F.

==================================================
1. PROTEGER LA PRIMERA REVISIÓN C1
==================================================

Antes de modificar:

- confirmar HEAD 4d9839f;
- confirmar schema v31;
- confirmar C1 activa;
- confirmar que la primera revisión C1 todavía no se ha publicado;
- comprobar si existe un request pendiente o scheduler programado;
- crear backup;
- calcular SHA-256;
- ejecutar integrity_check;
- ejecutar foreign_key_check;
- capturar baseline.

Si la primera revisión C1 todavía está WAITING_FOR_WINDOW:

- completar este hotfix antes de que el scheduler la publique.

Si ya se generó una revisión C1:

- no borrarla;
- no ejecutarla automáticamente;
- marcarla como no operativa si contiene recomendaciones menores de 2 USD;
- regenerarla idempotentemente después del hotfix;
- conservar lineage y auditoría.

No permitir una carrera entre despliegue y scheduler.

==================================================
2. REGLA DE MATERIALIDAD DE 2 USD
==================================================

En la cartera de crecimiento, una recomendación ejecutable no debe tener un
importe bruto absoluto inferior a:

MIN_EXECUTABLE_GROSS_NOTIONAL_USD =
2.00

Calcular:

recommended_gross_notional =
ABS(recommended_trade_quantity × certified_reference_price)

Aplicar el filtro después de:

- selección;
- cálculo de pesos;
- cálculo de target_quantity;
- cálculo del delta frente al holding real;

pero antes de:

- publicar recomendaciones ejecutables;
- mostrar acciones a Omar;
- contar compras o ventas;
- crear formularios de ejecución.

Si:

recommended_gross_notional < 2.00

no publicar:

- INCORPORAR;
- AUMENTAR;
- REDUCIR;
- ELIMINAR;

como operación ejecutable.

Clasificar internamente como:

NO_ACTION_BELOW_MIN_NOTIONAL

o usar el estado canónico equivalente no ejecutable.

No mostrar un formulario de fill Wio para esa línea.

==================================================
3. SEMÁNTICA DEL UMBRAL
==================================================

El umbral se refiere al principal bruto estimado de la operación:

ABS(quantity × price)

No incluir:

- comisión;
- cash effect;
- slippage;
- estimación de tasas;

para decidir si alcanza los 2 USD.

La comisión debe seguir separada.

Comparar usando precisión decimal completa.

No redondear primero a dos decimales para decidir.

Ejemplos:

gross =
1.9999
→ no ejecutable

gross =
2.0000
→ ejecutable

gross =
2.004
→ ejecutable

El valor mostrado puede redondearse después.

==================================================
4. TRATAMIENTO DEL DELTA NO EJECUTADO
==================================================

Si una compra o aumento queda por debajo de 2 USD:

- no comprar;
- conservar el importe como cash;
- no redistribuirlo de forma iterativa sin una regla explícita;
- registrar la razón de materialidad.

Si una reducción queda por debajo de 2 USD:

- no vender;
- conservar la cantidad actual;
- registrar la razón.

Si una eliminación completa tiene un valor inferior a 2 USD:

- no generar una operación ejecutable;
- clasificarla como residual no material;
- no ocultar que la posición residual existe.

No inventar una cantidad para alcanzar artificialmente los 2 USD.

No redondear cantidades al alza para superar el umbral.

No modificar holdings por el filtro.

==================================================
5. CONFIGURACIÓN Y LINEAGE
==================================================

El umbral debe estar versionado dentro de la metodología C1 o de su
configuración operativa sellada:

minimum_executable_gross_notional =
2.00

currency =
USD

No hardcodear el valor únicamente en la UI.

El backend es la autoridad.

La UI debe reflejar la misma regla.

Persistir por línea filtrada:

- ticker;
- action_before_materiality_filter;
- recommended_quantity_before_filter;
- reference_price;
- gross_notional;
- threshold;
- resulting_status;
- reason = BELOW_MIN_EXECUTABLE_NOTIONAL.

No alterar C0 histórico.

No modificar revisiones históricas.

==================================================
6. EFECTO EN EL RESUMEN DE LA REVISIÓN
==================================================

La revisión C1 debe mostrar:

- recomendaciones ejecutables;
- líneas no ejecutables por materialidad;
- cash que permanece sin desplegar por el umbral;
- número de líneas filtradas;
- importe bruto total filtrado.

Añadir al resumen:

BELOW_MIN_NOTIONAL_LINES =
<número>

BELOW_MIN_NOTIONAL_TOTAL_GROSS =
<importe>

REMAINING_CASH_DUE_TO_MATERIALITY =
<importe>

No presentar el cash residual como error de sizing.

==================================================
7. AUDITORÍA DEL BENCHMARK
==================================================

Auditar el panel que mostró, para el 21 de julio de 2026:

CARTERA =
+4,31 %

S&P 500 =
+0,34 %

Determinar cuál de estos casos aplica:

CASO A — BUG REAL

El +4,31 % de la cartera es acumulado desde el inicio del periodo, pero el
+0,34 % del S&P 500 es únicamente la variación diaria del 21 de julio.

Esto mezcla:

cumulative portfolio return
contra
daily benchmark return

y debe corregirse.

CASO B — COMPARACIÓN CORRECTA

Ambos valores son acumulados desde la misma fecha base y el +0,34 % es el
rendimiento acumulado del benchmark durante el mismo periodo.

No asumir el resultado.

Trazar el código, consultas, fuente de precios y transformaciones.

Entregar:

BENCHMARK_COMPARISON_CASE =
A_BUG /
B_CORRECT

con evidencia reproducible.

==================================================
8. MISMA BASE TEMPORAL
==================================================

Para una comparación correspondiente a julio de 2026, la fecha base debe ser:

BENCHMARK_BASE_DATE =
2026-06-30

Usar el cierre del 30 de junio de 2026 como denominador.

No utilizar el cierre del 1 de julio como base.

La primera observación de julio debe representar el rendimiento desde:

close 2026-06-30
hasta
close 2026-07-01

Para cualquier fecha t de julio:

benchmark_cumulative_return(t) =
benchmark_value(t) / benchmark_value(2026-06-30) - 1

La serie de cartera debe utilizar una base económica equivalente:

portfolio_cumulative_return(t) =
portfolio_return_index(t) / portfolio_return_index(2026-06-30) - 1

En la fecha base:

portfolio_cumulative_return(2026-06-30) =
0

benchmark_cumulative_return(2026-06-30) =
0

==================================================
9. MISMO MÉTODO DE RETORNO
==================================================

La comparación debe ser homogénea.

Permitido:

A. acumulado contra acumulado;

o:

B. diario contra diario.

El panel de rendimiento del periodo debe utilizar:

ACUMULADO CONTRA ACUMULADO

No mezclar:

- TWR acumulado de cartera con retorno diario del benchmark;
- variación diaria de la cartera con benchmark acumulado;
- bases de fechas diferentes;
- cierre anterior distinto;
- porcentajes ya acumulados acumulados otra vez.

Si existen aportaciones o retiradas durante el periodo:

- utilizar la serie canónica de rentabilidad de la cartera neutral a flujos
  externos;
- no utilizar simplemente la variación bruta del NAV;
- mantener la aportación neutral para el rendimiento.

Documentar si el benchmark representa:

- price return;
- total return;
- ETF proxy;

y cuál es su fuente.

No cambiar silenciosamente de índice o instrumento durante el hotfix.

==================================================
10. ALINEACIÓN DE FECHAS
==================================================

Alinear cartera y benchmark por sesiones bursátiles comparables.

Resolver correctamente:

- fines de semana;
- festivos;
- valores ausentes;
- zonas horarias;
- fecha de evidencia;
- cierre oficial.

No hacer forward-fill desde una fecha posterior.

No utilizar un precio intradía para una serie de cierres.

Si falta el cierre del benchmark para una sesión:

- marcar dato pendiente;
- no comparar contra cero;
- no reutilizar la variación diaria anterior;
- no publicar un exceso de retorno incorrecto.

==================================================
11. EXCESO DE RETORNO
==================================================

Calcular para cada fecha t:

relative_performance(t) =
portfolio_cumulative_return(t)
-
benchmark_cumulative_return(t)

Solo después de construir ambas series homogéneas.

Para el 21 de julio de 2026, entregar:

PORTFOLIO_BASE_VALUE =
<valor>

PORTFOLIO_2026_07_21_VALUE =
<valor>

PORTFOLIO_CUMULATIVE_RETURN =
<porcentaje>

BENCHMARK_BASE_CLOSE_2026_06_30 =
<valor>

BENCHMARK_CLOSE_2026_07_21 =
<valor>

BENCHMARK_CUMULATIVE_RETURN =
<porcentaje>

BENCHMARK_DAILY_RETURN_2026_07_21 =
<porcentaje>

RELATIVE_PERFORMANCE =
<porcentaje>

Esto debe demostrar inequívocamente si el +0,34 % era diario o acumulado.

==================================================
12. TESTS DEL MÍNIMO DE 2 USD
==================================================

Crear tests para:

A. compra de 1,99 USD → no ejecutable;
B. compra de 2,00 USD → ejecutable;
C. compra de 2,01 USD → ejecutable;
D. venta de 1,99 USD → no ejecutable;
E. eliminación residual inferior a 2 USD → no ejecutable;
F. importe usa precisión completa antes de redondear;
G. comisión no ayuda a alcanzar el mínimo;
H. filtro no modifica holdings;
I. filtro no crea movimiento;
J. filtro no crea formulario Wio;
K. cash residual permanece como cash;
L. resumen informa líneas filtradas;
M. backend y UI usan el mismo umbral;
N. C0 histórico no cambia;
O. C1 sigue usando investable_nav.

==================================================
13. TESTS DEL BENCHMARK
==================================================

Añadir como mínimo estas invariantes:

A. FECHA BASE

En 2026-06-30:

portfolio_series =
0 exactamente

benchmark_series =
0 exactamente

No aceptar tolerancia que oculte un desfase de fecha.

B. MISMO MÉTODO

Para cada punto comparado:

portfolio_return_type =
benchmark_return_type =
CUMULATIVE

C. MISMA FECHA BASE

portfolio_base_date =
benchmark_base_date =
2026-06-30

D. PRIMER DÍA

El valor del 1 de julio usa el cierre del 30 de junio como denominador.

E. CASO SINTÉTICO

Base:

portfolio =
100

benchmark =
100

Día 1:

portfolio =
104.31

benchmark =
100.34

Resultado:

portfolio cumulative =
4.31 %

benchmark cumulative =
0.34 %

relative =
3.97 puntos porcentuales

F. DETECCIÓN DE MEZCLA

Crear un fixture donde:

- benchmark diario = 0,34 %;
- benchmark acumulado = 2,10 %.

El test debe fallar si el panel compara el 4,31 % acumulado de la cartera
contra el 0,34 % diario.

G. FLUJOS EXTERNOS

Una aportación aumenta NAV, pero no altera la rentabilidad acumulada de la
cartera.

H. FECHAS AUSENTES

Una sesión ausente no se convierte en retorno cero ni en un valor diario
reutilizado.

==================================================
14. CORRECCIÓN SI APLICA CASO A
==================================================

Si se confirma CASO A:

- corregir el constructor de la serie;
- usar retorno acumulado del benchmark desde 2026-06-30;
- recalcular los puntos del periodo;
- corregir relative performance;
- corregir tarjetas, gráfica y tooltips;
- corregir API y UI;
- invalidar cualquier caché derivada incorrecta;
- no modificar precios fuente;
- documentar el incidente.

No sobrescribir snapshots históricos sin auditoría.

Si se persisten métricas derivadas:

- regenerarlas de forma versionada;
- conservar evidencia before/after.

==================================================
15. SI APLICA CASO B
==================================================

Si se confirma CASO B:

- no modificar la fórmula;
- añadir igualmente las invariantes;
- documentar la evidencia;
- mejorar etiquetas para indicar:

“Rendimiento acumulado desde 30/06/2026”

en ambas series.

Evitar que el usuario pueda confundir el dato con variación diaria.

==================================================
16. UI
==================================================

Recomendaciones inferiores a 2 USD:

- no aparecen como operaciones ejecutables;
- pueden mostrarse en un apartado informativo:
  “Ajustes no ejecutables por importe inferior a 2 USD”;
- no tienen botón para registrar ejecución Wio.

Panel comparativo:

- mostrar periodo;
- mostrar fecha base;
- indicar “acumulado”;
- mostrar fecha de último cierre;
- diferenciar claramente:
  - rendimiento del periodo;
  - variación diaria;
  - exceso de retorno.

No mostrar “S&P 500 +0,34 %” sin contexto temporal.

==================================================
17. REGRESIONES Y SEGURIDAD
==================================================

Ejecutar:

- Growth C1;
- contribution-to-recalculation;
- recommendation engine;
- Book;
- holdings;
- cash;
- NAV;
- rentabilidad neutral a aportaciones;
- benchmark;
- dashboard;
- scheduler;
- startup catch-up;
- migration;
- repository integrity.

Requisitos de tests nuevos:

PASS > 0
FAIL = 0
NO_RESULT = 0

No modificar:

- fills Wio;
- movimientos;
- holdings;
- cash;
- aportaciones;
- revisiones históricas;
- C0;
- activación C1;
- Gate 2F.

==================================================
18. DESPLIEGUE
==================================================

Antes:

- detener carrera con la primera revisión C1;
- backup;
- SHA-256;
- integrity_check;
- foreign_key_check;
- baseline;
- preview de la primera revisión C1 con el filtro de 2 USD;
- preview de la comparación del benchmark;
- tests;
- Chrome.

Después:

- desplegar backend y UI;
- reiniciar una sola instancia;
- comprobar /ping;
- comprobar schema v31;
- comprobar C1 activa;
- reanudar scheduler;
- verificar que la primera revisión C1 usa el filtro;
- verificar benchmark.

No crear aportaciones.
No crear órdenes Wio.
No hacer push.
No crear tag.

==================================================
19. COMMIT
==================================================

Crear un único commit local:

fix(growth): enforce trade materiality and align benchmark returns

==================================================
20. ENTREGA
==================================================

CANONICAL_HEAD_BEFORE =
4d9839f

CANONICAL_HEAD_AFTER =
<hash>

PRODUCTION_SCHEMA =
v31

C1_STATUS =
ACTIVE

FIRST_C1_REVIEW_STATUS_BEFORE =
<estado>

FIRST_C1_REVIEW_STATUS_AFTER =
<estado>

MIN_EXECUTABLE_GROSS_NOTIONAL_USD =
2.00

BELOW_MIN_NOTIONAL_RECOMMENDATIONS_PUBLISHED =
0

BELOW_MIN_NOTIONAL_LINES =
<número>

BELOW_MIN_NOTIONAL_TOTAL_GROSS =
<importe>

REMAINING_CASH_DUE_TO_MATERIALITY =
<importe>

C0_HISTORICAL_UNCHANGED =
PASS/FAIL

C1_INVESTABLE_NAV_UNCHANGED =
PASS/FAIL

BENCHMARK_COMPARISON_CASE =
A_BUG /
B_CORRECT

PORTFOLIO_BASE_DATE =
2026-06-30

BENCHMARK_BASE_DATE =
2026-06-30

PORTFOLIO_RETURN_METHOD =
CUMULATIVE

BENCHMARK_RETURN_METHOD =
CUMULATIVE

BASE_DATE_PORTFOLIO_RETURN =
0

BASE_DATE_BENCHMARK_RETURN =
0

PORTFOLIO_CUMULATIVE_RETURN_2026_07_21 =
<porcentaje>

BENCHMARK_CUMULATIVE_RETURN_2026_07_21 =
<porcentaje>

BENCHMARK_DAILY_RETURN_2026_07_21 =
<porcentaje>

RELATIVE_PERFORMANCE_2026_07_21 =
<porcentaje>

BENCHMARK_API =
PASS/FAIL

BENCHMARK_UI =
PASS/FAIL

BENCHMARK_INVARIANT_TESTS =
PASS/FAIL

MATERIALITY_TESTS =
PASS/FAIL

HISTORICAL_REVIEWS_UNCHANGED =
PASS/FAIL

EXISTING_MOVEMENTS_UNCHANGED =
PASS/FAIL

EXISTING_HOLDINGS_UNCHANGED =
PASS/FAIL

PRODUCTION_CONTRIBUTIONS_CREATED =
0

PRODUCTION_BROKER_ORDERS_SENT =
0

GATE_2F_REMAINS_INACTIVE =
PASS/FAIL

CHROME =
PASS/FAIL

MIZAN_JAVASCRIPT_ERRORS =
<número>

SECRET_SCAN =
PASS/FAIL

NO_PUSH_PERFORMED =
PASS/FAIL

NO_TAG_CREATED =
PASS/FAIL

LOCAL_COMMIT =
<hash>

EXACT_REMAINING_BLOCKERS =
NONE/<lista>

Detenerse después de desplegar y verificar el hotfix.