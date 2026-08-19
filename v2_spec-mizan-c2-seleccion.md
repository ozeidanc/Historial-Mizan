## Contrato de checks y factores C2

El motor dispone de 15 checks binarios, pero C2 utiliza 11 factores continuos
para el ranking.

La distinción es normativa:

- los checks binarios se conservan para elegibilidad, explicación, auditoría
  y compatibilidad;
- los factores continuos se utilizan para ordenar;
- no todo check binario es un factor de ranking;
- una tasa de verdes alta no implica que la magnitud continua subyacente
  carezca de capacidad discriminante.

### Checks binarios preservados

1. `pe_hist`
2. `pe_sect`
3. `below_tgt`
4. `debt`
5. `margins`
6. `fcf`
7. `eps_rev`
8. `rev_grow`
9. `surprise`
10. `ma200`
11. `mcap`
12. `coverage`
13. `insider`
14. `piotroski`
15. `dividend`

C0 y C1 mantienen intacto el uso actual de estos checks.

### Factores continuos de ranking C2

C2 utiliza inicialmente, con pesos iguales:

1. `pe_hist`
2. `pe_sect`
3. `below_tgt`
4. `debt`
5. `margins`
6. `fcf`
7. `eps_rev`
8. `rev_grow`
9. `surprise`
10. `ma200`
11. `piotroski`

Para cada factor deben definirse y sellarse:

- valor crudo;
- semántica económica;
- dirección;
- unidad;
- fuente;
- fecha económica;
- `acceptedDate`;
- política point-in-time;
- tratamiento de valores no finitos;
- winsorización;
- percentile rank;
- missing policy.

No convertir `g/a/r` en una escala numérica artificial. El continuo debe
proceder de la magnitud económica original.

### Checks que no forman parte del compuesto continuo

#### `mcap`

Rol C2:

`ELIGIBILITY_GATE`

La capitalización se utiliza como requisito de elegibilidad y para los
análisis de sensibilidad del universo. No recibe peso adicional en el score,
para no introducir silenciosamente una preferencia por tamaño después de
haber aplicado ya el umbral mínimo.

#### `coverage`

Rol C2:

`DATA_SUFFICIENCY_GATE`

Mide si existen inputs suficientes. No representa una característica
económica de la empresa y no forma parte del score.

#### `insider`

Rol C2 inicial:

`DISPLAY_AND_DIAGNOSTIC_ONLY`

Se excluye del compuesto por su cobertura observada aproximada del 3 %.

Puede reconsiderarse en una metodología posterior si existe una fuente
point-in-time con cobertura suficiente.

#### `dividend`

Rol C2 inicial:

`DISPLAY_AND_DIAGNOSTIC_ONLY`

Se excluye por:

- cobertura parcial;
- semántica discutible dentro de una estrategia de crecimiento;
- riesgo de introducir indirectamente un sesgo de estilo no definido.

No se elimina de la UI ni del archivo PIT.

### Factores binariamente saturados

No retirar un factor del score continuo únicamente porque su check binario
tenga una tasa de verdes superior al 90 % o inferior al 10 %.

Evaluar por separado:

- discriminación binaria;
- dispersión de la magnitud continua;
- cobertura;
- estabilidad;
- procedencia point-in-time;
- correlación;
- contribución al ranking.

En particular, permanecen inicialmente como factores continuos:

- `rev_grow`;
- `below_tgt`;
- `debt`;
- `fcf`;
- `surprise`.

La evidencia observada demuestra que `rev_grow`, aunque es verde para todas
las empresas que superan el gate, conserva dispersión económica dentro del
universo elegible y resuelve empates que el check binario no puede resolver.

### Doble rol de `rev_grow`

`rev_grow` tiene dos roles expresamente autorizados:

1. gate de crecimiento mínimo;
2. factor continuo de ranking.

Gate:

`revGrowthPct >= 0.08`

Ranking:

percentile rank transversal de `revGrowthPct` dentro del universo elegible,
con dirección `HIGHER_IS_BETTER`.

El gate define el mínimo requerido por la estrategia. El factor continuo
ordena a las empresas que ya superaron ese mínimo.

### Cobertura mínima

Número de factores continuos aprobados:

`11`

Cobertura mínima:

`80 %`

Regla determinista:

`MINIMUM_CONTINUOUS_FACTORS_PRESENT = 9`

Si están presentes al menos nueve factores:

- el ticker puede permanecer elegible;
- cada factor individual ausente recibe exactamente `0.5`;
- se registra el motivo de ausencia.

Si están presentes ocho factores o menos:

`ELIGIBILITY_STATUS = INSUFFICIENT_FACTOR_COVERAGE`

`coverage` no se incorpora como factor adicional al compuesto.

### Compuesto inicial

Para cada factor:

1. obtener la magnitud cruda point-in-time;
2. winsorizar transversalmente en P1/P99;
3. convertir a percentile rank `[0,1]`;
4. invertir la dirección cuando `LOWER_IS_BETTER`;
5. asignar `0.5` si falta el factor y se supera la cobertura mínima;
6. calcular la media aritmética simple de los 11 valores.

Todos los factores tienen inicialmente el mismo peso.

No optimizar pesos en esta iteración.

### Correlación

La correlación no provoca la retirada automática de factores durante esta
iteración.

Debe reportarse especialmente:

- `pe_hist` frente a `pe_sect`;
- factores de valoración frente a momentum;
- `eps_rev` frente a `ma200`;
- `rev_grow` frente a márgenes y FCF.

La posible redundancia de factores se estudia durante la sombra forward. No
se reajustan pesos ni se eliminan factores utilizando ocho sesiones
históricas o un backtest no point-in-time.

### Contrato de salida

El snapshot y audit trail deben distinguir:

`binary_checks`

`continuous_raw_values`

`continuous_percentiles`

`continuous_factor_coverage`

`continuous_composite_score`

`eligibility_gates`

`display_only_checks`

Esto impide confundir nuevamente los 15 checks con los 11 factores de
ranking.