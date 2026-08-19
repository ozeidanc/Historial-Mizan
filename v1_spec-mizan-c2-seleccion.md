# Especificación — Motor de selección Mizan: diagnóstico y metodología C2

> **Para Copilot:** este documento tiene dos partes con propósitos distintos. La **Parte A** es el diagnóstico y debe llegar íntegra a Claude Code para que entienda *por qué* se cambia cada cosa; sin ella tomará decisiones de implementación equivocadas. La **Parte B** son las restricciones de arquitectura y tampoco debe resumirse: es lo que impide romper producción. Las fases 0 a 3 puedes reformatearlas como prefieras, pero **no reordenes las fases**: el orden es una dependencia, no una preferencia.
>
> Referencias de archivo y línea provienen de una auditoría read-only sobre `HEAD productivo = 1d67c54`, `SCHEMA = v31`.

---

# PARTE A · Diagnóstico

Metodología activa: `GROWTH_C1_OPERATIONAL_NAV`. La selección es el motor C0 compartido (`methodology_hash = 103b4256f26d9e94c7c64730cd9e04c2953119962d2e246b2aef9d8302854182`). C1 solo cambia la base de sizing.

## A.1 El score no tiene resolución suficiente para ordenar

`score = greens / total`, con totales entre 11 y 14 (`calcularChecks`, server.js:1849-2232; gate en `portfolio-selection-engine.mjs:44-46`).

Consecuencia aritmética: existen ~15 valores posibles de score en todo el universo, y **un check que pase de verde a ámbar mueve el score 1/14 = 7,1 puntos porcentuales.**

Evidencia de la sesión 8 (2026-07-27):

| Ticker | Score | Rank |
|---|---|---|
| AAPL | 13/14 = 0,929 | 1 |
| ADI | 12/14 = 0,857 | 4 |
| PAYX | 11/14 = 0,786 | 24 |
| PANW | 10/13 = 0,769 | 26 (fuera) |

Del rank 1 al rank 24 hay **0,143 de recorrido total**, y un check vale 0,071. Es decir: **los 24 primeros puestos de la cartera caben en aproximadamente dos escalones de score, y un solo check equivale a ~12 posiciones de ranking.**

De aquí derivan tres fallos:

**A.1.a — Ordenación alfabética de facto.** Con grupos de empate enormes, el desempate `ticker_asc` (`portfolio-selection-engine.mjs:60`) decide plazas reales de cartera. Indicio fuerte en los datos: AAPL es rank 1 y ADI es rank 4, y alfabéticamente entre ambos están exactamente ABNB y ADBE. La cartera actual tiene 9 de 25 posiciones empezando por A. **Hay que medirlo antes de asumirlo, pero si se confirma, es el fallo más grave del sistema.**

**A.1.b — Denominadores no comparables.** PANW es 10/**13** y PAYX es 11/**14**. Los `na` se excluyen del total, así que un valor con menos checks evaluados compite con una fracción distinta. **La disponibilidad de datos está decidiendo la composición de la cartera.** Si a PANW se le hubiera evaluado ese decimocuarto check y saliera verde, sería 11/14: empatado con PAYX.

**A.1.c — La binarización destruye la magnitud.** Una empresa con P/E 15 contra mediana sectorial de 30 recibe el mismo verde que otra con P/E 29. Toda la información sobre *cuánto* mejor está cada factor se descarta en el momento de convertirlo a g/a/r. Es precisamente donde vive la capacidad de discriminación del modelo.

## A.2 La implementación contradice su propio sello

El sello indica `scheduled_review_frequency: trimestral`, y la auditoría lo confirma: *"el rebalanceo canónico es trimestral; entre rebalanceos el top-25 congelado no debería cambiar salvo re-freeze."*

Pero el scheduler ejecuta `nightly /internal/freeze-c0-selection` (server.js:4849), y `seleccionar()` (`portfolio-selection-engine.mjs:51-87`) es un **recálculo duro y sin estado**: gates → sort → `slice(0,25)`, sin recibir la selección previa como input. `TURNOVER_BUFFER = NONE`.

Resultado observado: PANW entró y salió en sesiones consecutivas por una diferencia de score de 1,7 pp, con una pérdida realizada de ~4,83 USD sobre una posición de ~110 USD. **La amortiguación que el sello prevé no se está aplicando a los cambios de composición.**

## A.3 Puede no haber selección real que optimizar

`UNIVERSE_CONSTRUCTION` = 123 curadas estáticas (`COMPANIES.nasdaq(101) + COMPANIES.dow(22)`, server.js:2447-2462), pre-filtradas por sector en `construirUniverso` (:2624) y de nuevo en `computeC0Provenance` (:4661).

Filtros de elegibilidad en cadena: sector ∈ `technolog|consumer cyclical|discretionary` → `rev_grow === "g"` → `revGrowthPct ≥ 0,08` → `score ≥ 7/11` → `slice(0,25)`.

**Si tras los filtros quedan ~30 elegibles y se seleccionan 25, el modelo está comprando el 83% de su universo elegible.** Eso no es selección: es el filtro. Y en ese caso el alfa de selección es cero por construcción, independientemente de la calidad del score. Explicaría por qué la cartera se comporta como un índice tecnológico equiponderado.

**Esta es la medición más importante del sistema y hay que hacerla antes de cambiar nada.**

## A.4 Posible peso muerto en los factores

`mcap = capitalización > mínimo`. En un universo Nasdaq-100 + Dow-22 es previsible que lo pase ~el 100%. Un check que casi todos aprueban no aporta discriminación pero suma 1 al numerador y al denominador de todos, comprimiendo las diferencias entre el resto.

## A.5 Los backtests no son fiables hoy

Observación 3 de la auditoría: solo `revGrowthPct` es PIT (por `acceptedDate ≤ cutoff`); el score de 7/11 usa `calcularChecks` con datos vivos/anuales. **El gate de score no es estrictamente point-in-time.**

Implicación crítica: **cualquier backtest comparando metodologías está contaminado con look-ahead bias.** Esto crea una dependencia dura sobre el plan de validación (ver Fase 2).

---

# PARTE B · Restricciones de arquitectura (NO NEGOCIABLES)

Estas reglas tienen prioridad sobre cualquier otra instrucción del documento. Ante conflicto o duda, **para y pregunta**.

1. **No mutar C0 ni C1.** El `methodology_hash` es el ancla de auditoría de todas las sesiones históricas. Los cambios van en una **metodología nueva**: `GROWTH_C2_*`, con su propio archivo sellado (`crecimiento-v2-c2-sealed.mjs`) y su propio hash. `crecimiento-v2-c0-sealed.mjs` no se toca.

2. **Modo sombra obligatorio.** C2 calcula, registra y se audita, pero **no genera recomendaciones ejecutables** hasta ser promovida explícitamente. `generateDailyRecommendationSession` (server.js:5658) sigue leyendo la selección activa de C0/C1.

3. **`seleccionar()` debe seguir siendo una función pura.** Cuando necesite la selección previa (histéresis), se le pasa como **parámetro explícito** (`seleccionPrevia`), nunca leyéndola de BD dentro de la función. La testabilidad de esa función es un activo del sistema.

4. **Append-only intacto.** `prospective_data_audit` conserva su semántica. Añade un `kind='C2_SELECTION_SHADOW'` nuevo; no reutilices ni modifiques `C0_SELECTION_FROZEN`.

5. **Test dorado de no-regresión.** Tras los cambios, re-ejecutar sesiones históricas con C0 seleccionada debe reproducir **byte-idénticamente** lo registrado en el audit trail, incluido el `methodology_hash`. Este test es la prueba de que no se ha roto nada y es criterio de aceptación bloqueante.

6. **Prohibido refactorizar.** No renombres, no reorganices, no actualices dependencias, no "limpies" código que funciona. Si detectas algo grave, anótalo en un informe aparte.

7. **Un cambio por commit, aislado y medible.** No agrupes.

8. **Rama `feature/c2-continuous-scoring`.** Ningún commit a main.

9. **Prohibido optimizar pesos de factores en esta iteración.** C2 arranca con **pesos iguales**, para poder atribuir el efecto a la normalización y no a un reajuste simultáneo. Cualquier optimización de pesos es una iteración posterior y exige validación out-of-sample. Con 19 sesiones de historia, ajustar pesos es sobreajuste garantizado.

---

# FASE 0 · Medición (read-only, antes de escribir código de producción)

Cinco consultas sobre datos existentes. **Entrega los resultados y espera confirmación antes de pasar a la Fase 1.** Estos números pueden cambiar qué merece la pena implementar.

**0.1 · Embudo de elegibilidad.** Para las últimas 20 sesiones, cuenta los supervivientes en cada etapa:

```
123 curadas
 → tras datos_sospechosos
 → tras gate sectorial (technolog|consumer cyclical|discretionary)
 → tras rev_grow === "g"
 → tras revGrowthPct ≥ 0,08
 → tras score ≥ 7/11
 → top-25
```

**Métrica clave: `elegibles / 25`.** Si la mediana está por debajo de 1,5, el sistema no está seleccionando y la prioridad pasa a ser la ampliación de universo, no el score.

**0.2 · Grupos de empate y desempate alfabético.** Para cada sesión: distribución de tamaños de grupo con `(greens, total)` idénticos, y **cuántas de las 25 plazas quedaron determinadas por `ticker_asc`**. Reporta también cuántos tickers de la cartera empiezan por A-C frente a lo esperado dado el universo.

**0.3 · Tasa de verdes por factor.** Para los 11 factores, porcentaje de verdes sobre el universo elegible. Marca los que superen el 90% o queden bajo el 10%: no discriminan.

**0.4 · Estabilidad del ranking.** Serie histórica del rank de cada ticker. Cuenta cruces de la frontera 25/26 por mes, e identifica qué tickers oscilan.

**0.5 · Correlación entre checks.** Matriz de correlación entre los 11 checks sobre el universo. Interesa especialmente si los de valor (`pe_hist`, `pe_sect`, `below_tgt`) correlacionan negativamente con los de momentum (`eps_rev`, `surprise`, `ma200`): indicaría que el conteo de verdes premia a la empresa mediocre en todo sobre la excelente en algo.

---

# FASE 1 · Metodología C2 (modo sombra)

Cambios ordenados por relación impacto/riesgo. **Cada uno en su propio commit, con su propio test.**

## 1.1 · Respetar la frecuencia trimestral (coste cero, impacto inmediato)

Separa dos conceptos que hoy están fusionados:

- **`computed_selection`** — se calcula cada noche y se registra siempre, para observación y auditoría. No ejecuta.
- **`active_selection`** — la que consume `generateDailyRecommendationSession`. Solo se sustituye por `computed_selection` cuando: (a) es fecha de rebalanceo trimestral programado, (b) salta una regla dura de riesgo explícitamente definida y documentada, o (c) hay override manual con justificación registrada.

Beneficio secundario: registrar el `computed_selection` diario sin ejecutarlo te da **la medida exacta de cuánto habría rotado el sistema**, gratis y sin coste de transacción.

Esto no es un cambio de metodología: es alinear la implementación con el sello. Puede aplicarse a C0/C1 sin cambiar el `methodology_hash`, **siempre que se confirme que el sello no exige el re-freeze nocturno.** Verifícalo antes.

## 1.2 · Score continuo (el cambio de mayor impacto)

Sustituye `score = greens/total` por un compuesto continuo. Mantén los checks binarios: siguen siendo útiles para la UI y para los gates de elegibilidad. Lo que cambia es **sobre qué se ordena**.

Para cada uno de los 11 factores:

1. **Valor crudo continuo.** Ejemplos: `pe_hist` → `(mediana_5a − pe) / mediana_5a`; `below_tgt` → `(target − precio) / precio`; `ma200` → `(precio − ma200) / ma200`. Documenta la fórmula y la dirección de cada uno.
2. **Winsorizar** en los percentiles 1 y 99 sobre el universo elegible de esa fecha, para que un outlier de datos no domine el ranking.
3. **Normalización transversal:** convierte a **rango percentil [0,1]** dentro del universo elegible de esa fecha. Usa percentil, no z-score: es más robusto a las colas gruesas típicas de datos financieros.
4. **Datos ausentes:** asigna el valor neutro **0,5**, nunca exclusión. Esto elimina de raíz el problema A.1.b de denominadores no comparables.
5. **Compuesto:** media aritmética simple de los 11 percentiles. **Pesos iguales en esta iteración** (ver restricción B.9).

Resultado: score continuo en [0,1] con resolución efectivamente infinita. Los empates desaparecen, el desempate alfabético deja de tener efecto práctico, y un cambio pequeño en un factor produce un cambio pequeño en el ranking en lugar de un salto de 12 puestos.

**Los gates de elegibilidad no cambian** en esta fase: sector, `rev_grow === "g"`, `revGrowthPct ≥ 0,08` y `score_binario ≥ 7/11` siguen aplicándose igual. Solo se cambia la clave de ordenación. Así el efecto es atribuible.

## 1.3 · Desempate económico (restaura la intención del sello)

La auditoría señala que `revGrowthPct_desc` está sellado como 3ª clave de desempate pero **no implementado** (código muerto; los empates van a `ticker_asc`). Impleméntalo: `sort(score_continuo_desc → revGrowthPct_desc → ticker_asc)`.

Requiere cargar `revGrowthPct` en las filas del ranking, que hoy no se cargan. Esto **restaura el comportamiento sellado**, no lo cambia.

## 1.4 · Banda tampón (histéresis)

- **Entrada:** `rank ≤ 25`
- **Salida:** solo si `rank > 30` (parámetro `exit_rank_buffer`, configurable)
- Si un incumbente está entre 26 y 30, **se mantiene**; si hay más de 25 tras aplicar la regla, se resuelve expulsando al de peor rank.

Implementación: `seleccionar(universo, config, seleccionPrevia)` — la selección previa entra **como parámetro**, respetando B.3. Cuando `seleccionPrevia` es `null` (primer arranque, o backtest desde cero), el comportamiento debe ser idéntico al actual.

## 1.5 · Banda de deriva en el sizing

`driftBandRel` está hoy en `null`, razón por la que `MANTENER` tiene 0 casos históricos y todo incumbente recibe un micro-ajuste. Es un parámetro previsto y sin asignar.

Fija `driftBandRel = 0.20`: no se opera mientras el peso esté dentro del ±20% relativo del objetivo (es decir, entre 3,2% y 4,8% sobre un objetivo del 4%). Complementa —no sustituye— el `minimum_executable_gross_notional = 2.00` ya activo en C1.

Efecto esperado: la mayoría de las 24 órdenes de mantenimiento diarias desaparecen y `MANTENER` pasa a ser el estado normal de un incumbente.

## 1.6 · Retirar factores sin capacidad discriminante

Según el resultado de 0.3: cualquier factor con tasa de verdes >90% o <10% sale del **compuesto de ranking**. Puede permanecer en el display y en los gates si aporta información al usuario, pero no debe diluir la ordenación.

---

# FASE 2 · Validación

**Bloqueante: sin esta fase, C2 no se promueve.**

## 2.1 · El problema del PIT (leer antes de planificar el backtest)

Según A.5, el score usa datos vivos y solo `revGrowthPct` es PIT. **Un backtest de C0 vs C2 sobre histórico está contaminado con look-ahead bias y no sirve para decidir.**

Dos caminos. Evalúa cuál es viable y propón uno:

- **Camino A — arreglar el PIT.** Reconstruir `calcularChecks` para que todos los factores usen datos con `acceptedDate ≤ cutoff`. Es el camino correcto y habilita backtesting fiable de forma permanente. Coste alto: requiere histórico de fundamentales con fechas de publicación.
- **Camino B — sombra forward.** Ejecutar C2 en paralelo sin operar, y comparar composición y retorno teórico durante 3-6 meses. Limpio y sin sesgo, pero lento.

Si ninguno es viable a corto plazo, **dilo explícitamente** y no presentes un backtest contaminado como evidencia.

## 2.2 · Comparativa C0 vs C2

Sobre el periodo y método que resulte de 2.1, reporta para ambas metodologías:

- Retorno acumulado y volatilidad realizada anualizada
- **Rotación de composición** (sustituciones por trimestre) y **rotación de sizing** (número e importe de órdenes)
- **Coste realizado de round-trips** cerrados
- Periodo medio de tenencia
- Solapamiento medio de composición entre sesiones consecutivas
- Diferencial contra tres benchmarks: SPY, QQEW y **el universo elegible equiponderado** (este último es el que aísla el alfa de selección)
- Número de plazas de cartera determinadas por desempate (debe ser ~0 en C2)

## 2.3 · Tests unitarios exigidos

- Percentil con datos ausentes → devuelve 0,5 exacto.
- Winsorización: un outlier extremo no altera el orden del resto.
- `seleccionar(..., seleccionPrevia=null)` reproduce el comportamiento sin histéresis.
- Histéresis: un incumbente en rank 27 se mantiene; en rank 31 sale.
- `driftBandRel`: una posición al 4,1% no genera orden; al 4,9% sí.
- **Test dorado (B.5):** re-ejecución de sesiones históricas bajo C0 reproduce byte-idénticamente el audit trail.

---

# FASE 3 · Promoción

No promuevas C2 salvo que se cumpla **todo** lo siguiente:

1. El test dorado pasa: C0 sigue reproduciéndose byte a byte.
2. La comparativa 2.2 está hecha con un método de validación no contaminado, o su contaminación está explícitamente cuantificada y aceptada por escrito.
3. La rotación de composición baja frente a C0 sin deterioro material del diferencial contra el universo elegible equiponderado.
4. Cero plazas de cartera determinadas por desempate.
5. Promoción tras feature flag, con capacidad de revertir a C0/C1 en una sola operación.

**Fuera de alcance en esta especificación** (posteriores, y solo tras promover C2): ampliación del universo a `roster.mjs` (~1.012), cupos por sector, y optimización de pesos de factores. Ampliar el universo **antes** de tener histéresis y score continuo empeoraría el churn: con más candidatas, la diferencia entre el puesto 25 y el 26 se vuelve microscópica.

---

# Entregables

1. Resultados de la Fase 0, **esperando confirmación** antes de continuar.
2. Un commit por cada cambio de la Fase 1, con sus tests.
3. Archivo sellado nuevo `crecimiento-v2-c2-sealed.mjs` y su `methodology_hash`, con C0 intacto.
4. Informe de validación de la Fase 2, incluyendo el dictamen sobre viabilidad del PIT.
5. Evidencia del test dorado.
6. Informe aparte con cualquier problema detectado y **no** tocado.

# Criterios de aceptación

- `crecimiento-v2-c0-sealed.mjs` sin modificar; `methodology_hash` de C0 inalterado.
- Test dorado en verde sobre al menos 10 sesiones históricas.
- C2 en modo sombra: cero recomendaciones ejecutables generadas por C2 antes de la promoción.
- `seleccionar()` sigue siendo pura (sin acceso a BD ni red).
- Ninguna dependencia nueva sin aprobación previa.
- Todo parámetro nuevo en configuración, ninguno hardcodeado.
