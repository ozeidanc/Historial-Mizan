# Especificación C2 · v2 — Motor de selección Mizan

> **Sustituye íntegramente a la v1.** La v1 nunca llegó a transmitirse y además contenía un error material: hablaba de 11 factores cuando el motor tiene 15 checks. Este documento corrige eso y consolida los hallazgos de la Fase 0 y de los checkpoints posteriores.
>
> **Para pegar como texto plano**, no como adjunto.

---

## 0 · Estado en el momento de escribir esto

| Elemento | Estado |
|---|---|
| Motor `sealed-tiebreak-compliant` | CONSTRUIDO, `READY_NOT_ACTIVE`, golden `6447af05` intacto, replay 8/8 |
| Activación prospectiva C0/C1 | PENDIENTE de autorización |
| Recomendación 168 (ELIMINAR PANW) | CONTENIDA, pendiente de decisión |
| `AS_OBSERVED` + PIT forward | AUTORIZADO, no construido |
| C2.0 / C2.1 | Bloqueados por este documento |
| `SECTOR_GATE` para C2.1 | NONE (aprobado) |
| `MINIMUM_FACTOR_COVERAGE` | 80% (aprobado) |

---

# PARTE A · Diagnóstico consolidado

Todos los puntos siguientes están **confirmados con evidencia**, no son hipótesis.

**A.1 · Resolución del score.** `score = greens/total`, totales 13-15. Solo ~6 de los 15 checks discriminan (pe_hist 39%, pe_sect 44%, piotroski 58%, eps_rev 60%, ma200 65%, margins 83%); el resto está al 100%, es gate de diseño o tiene cobertura insuficiente. Seis factores binarios producen siete escalones para 36 candidatas: ~5 por escalón. **El corte del top-25 cayó dentro de un empate en 8 de 8 sesiones**, con gap de score r25−r26 = 0,000 pp.

**A.2 · Denominadores variables.** `na` se excluye del total, así que se comparan fracciones con bases distintas. Afecta al ranking **y a la elegibilidad**: MELI y TXN (9/14 = 0,643) dejarían de superar el umbral 0,636 bajo denominador fijo con neutral.

**A.3 · Desempate no conforme.** RESUELTO. El sello exige `[score_desc, greens_desc, revGrowthPct_desc, ticker_asc]`; el código ejecutaba tres claves. Motor versionado ya construido. `ALPHABETIC_SEATS_AFTER_SEALED_TIEBREAK = 0`.

**A.4 · Cadencia.** `CADENCE_CLASSIFICATION = DAILY_FREEZE_CHANGED_OPERATIONAL_COMPOSITION`, trazado con IDs (freeze 108 → rec 130 → mov 91 → holding 167). El `methodology_hash` **incluye** `scheduled_review_frequency`, luego cualquier cambio de cadencia de decisión exige metodología nueva.

**A.5 · Caché de fundamentales WRITE_ONCE.** `EXISTING_POPULATED_CACHE_CAN_BE_REPLACED = NO`. Un ticker con datos nunca se actualiza. El riesgo acotado: tickers cuya caché es anterior a un filing posterior que WRITE_ONCE nunca capturó. La antigüedad de `mtime` **no** implica antigüedad económica.

**A.6 · Selectividad.** 36 elegibles para 25 plazas (ratio 1,44; 69,4% del universo elegible). Límite de diseño por universo estrecho, **no** prueba de ausencia de alfa.

**A.7 · Concentración sectorial.** `POLICY_IMPOSED`. El roster completo (1.060) es sectorialmente diverso (Healthcare 25,9% > Technology 22,4%). El `sector_regex` es la causa.

---

# PARTE B · Restricciones de arquitectura (NO NEGOCIABLES)

Prioridad sobre cualquier otra instrucción. Ante conflicto, **para y pregunta**.

1. **No mutar C0 ni C1.** Metodologías nuevas con archivo sellado y hash propios. Excepción única y ya validada: el arreglo de conformidad del desempate, que hace que el código cumpla el sello sin cambiar el hash.
2. **Modo sombra obligatorio** para C2.0 y C2.1. Cero recomendaciones ejecutables antes de promoción explícita.
3. **`seleccionar()` sigue siendo pura.** El estado previo entra como parámetro (`seleccionPrevia`), nunca por lectura de BD.
4. **Append-only intacto.** `kind` nuevo para las sombras; no reutilizar ni reescribir `C0_SELECTION_FROZEN`.
5. **Test dorado bloqueante.** Replay histórico bajo `legacy` byte-idéntico, hash `6447af05` incluido.
6. **Prohibido refactorizar.** Anota lo que veas mal en informe aparte; no lo toques.
7. **Un cambio por commit.**
8. **Prohibido optimizar pesos de factores.** C2.0 arranca con pesos iguales. Con menos de 60 sesiones, ajustar pesos es memorizar el pasado. Cualquier ponderación posterior exige validación out-of-sample.
9. **Prohibido presentar backtests contaminados como evidencia.** No existe historia PIT de fundamentales.

---

# PARTE C · CONTRATO DE FACTORES (§10 — esto es lo que desbloquea a Code)

Los 15 checks activos se particionan así. **Esta partición es normativa.**

## C.1 · CONTINUOUS_RANKING_FACTORS (11) — entran en el compuesto

| # | Check | Valor crudo continuo | Dirección |
|---|---|---|---|
| 1 | `pe_hist` | `(mediana_pe_5a − pe) / mediana_pe_5a` | mayor mejor |
| 2 | `pe_sect` | `(mediana_pe_sector − pe) / mediana_pe_sector` | mayor mejor |
| 3 | `below_tgt` | `(target_consenso − precio) / precio` | mayor mejor |
| 4 | `debt` | `netDebt / EBITDA` | menor mejor |
| 5 | `margins` | margen operativo TTM, y su delta YoY | mayor mejor |
| 6 | `fcf` | FCF yield = `FCF / mcap` | mayor mejor |
| 7 | `eps_rev` | variación % del target a 3 meses | mayor mejor |
| 8 | `rev_grow` | `revGrowthPct` (TTM, PIT por `acceptedDate`) | mayor mejor |
| 9 | `surprise` | sorpresa media de EPS en 4T, en % | mayor mejor |
| 10 | `ma200` | `(precio − ma200) / ma200` | mayor mejor |
| 11 | `piotroski` | puntuación 0-9 | mayor mejor |

**Advertencia normativa que anula una instrucción anterior:** un factor no discriminante en binario **puede ser altamente discriminante en continuo**, y esa es precisamente la señal que la binarización destruye. Caso demostrado: `rev_grow` es verde en el 100% de las elegibles (inútil como binario) y sin embargo su valor continuo resolvió 16 de 16 desempates. Lo mismo aplica a `debt`, `fcf`, `surprise` y `below_tgt`, todos por encima del 90% en verde. **Ninguno de los 11 se retira del compuesto por su tasa de verdes.**

## C.2 · GATE_ONLY_CHECKS (2) — no entran en el ranking

- **`mcap`** — umbral de elegibilidad. Nota para C2.1: al ampliar el universo a small/micro caps, `mcap` pasa a ser una **decisión de sesgo de tamaño**, no un factor de calidad. Se decide por política, no por score.
- **`coverage`** — meta-check sobre completitud de datos, no sobre la empresa. Alimenta el gate `MINIMUM_FACTOR_COVERAGE = 80%`.

## C.3 · EXCLUDED_CHECKS (2) — fuera del ranking y de los gates

- **`insider`** — cobertura del 3%. Por debajo de cualquier mínimo razonable. Mantener solo en display.
- **`dividend`** — cobertura del 56%, y semántica discutible en una metodología de crecimiento (una rentabilidad por dividendo alta correlaciona con crecimiento bajo, es decir, dirección contraria al eje). Solo display.

**Suma de control:** 11 + 2 + 2 = 15. ✅

---

# PARTE D · SEMÁNTICA DEL SCORE CONTINUO

Por cada uno de los 11 factores de C.1, en cada sesión:

1. **Valor crudo** según la fórmula de C.1. Documenta unidad y dirección.
2. **Winsorizar** en percentiles 1 y 99 sobre el universo elegible **de esa sesión**.
3. **Normalizar a rango percentil `[0,1]`** dentro del universo elegible de esa sesión. Percentil, no z-score: más robusto a las colas gruesas de datos financieros.
4. **Dato ausente → 0,5 exacto** (neutro). Nunca exclusión. Esto elimina de raíz el problema A.2 de denominadores.
5. **Compuesto = media aritmética simple de los 11 percentiles.** Pesos iguales (restricción B.8).

Resultado: score continuo en `[0,1]` con resolución efectivamente infinita. Los empates de frontera desaparecen y un cambio pequeño en un factor produce un cambio pequeño de rank, no un salto de 12 puestos.

**Los checks binarios permanecen** para display y para los gates de elegibilidad. Lo único que cambia es la clave de ordenación.

**Denominador fijo:** el `score_binario` que alimenta el gate `≥ 7/11` se recalcula sobre denominador fijo de 15 con neutral para ausentes. Consecuencia conocida y aceptada: MELI y TXN dejan de ser elegibles. Documéntalo en el changelog de metodología.

---

# PARTE E · HISTÉRESIS

- **Entrada:** `rank ≤ 25`
- **Salida:** solo si `rank > 30` (parámetro `exit_rank_buffer`, configurable)
- Incumbente entre 26 y 30 → se mantiene. Si tras aplicar la regla hay más de 25, sale el de peor rank.
- Firma: `seleccionar(universo, config, seleccionPrevia)`. Con `seleccionPrevia = null` el comportamiento debe ser idéntico al actual.

# PARTE F · SIZING

- `driftBandRel = 0.20` — no operar mientras el peso esté entre 3,2% y 4,8% sobre objetivo del 4%. Hoy está en `null`, razón por la que `MANTENER` tiene 0 casos históricos.
- Complementa, no sustituye, al `minimum_executable_gross_notional = 2.00` ya activo en C1.
- Referencia de coste: la compra de PANW pagó 0,52 USD de comisión sobre 110,80 = **0,47%**. Ida y vuelta, casi 1%.

---

# PARTE G · FASES

| Fase | Contenido | Depende de |
|---|---|---|
| **G.1** | `AS_OBSERVED` snapshot — sella lo que C0/C1 consume hoy, **antes** de cualquier refresco de fundamentales | Nada. Ejecutable ya |
| **G.2** | PIT forward archive | G.1 |
| **G.3** | Diagnóstico WRITE_ONCE: qué tickers filaron después de su `mtime` de caché | G.1 |
| **G.4** | C2.0 — score continuo + histéresis sobre el universo curado de 123, en sombra | Este documento |
| **G.5** | C2.1 — mismo motor sobre roster completo, sin gate sectorial, en sombra | G.4 + enriquecimiento |
| **G.6** | Arreglo del WRITE_ONCE e invalidación por antigüedad, y refresco controlado | G.1, G.2 y guarda de cadencia |

**C2.0 y C2.1 deben tener `methodology_hash` y lineage distintos.** Comparados entre sí aíslan exactamente una variable: el universo.

**Validación:** `ROUTE_5` (sombra forward, 3-6 meses). No existe historia PIT, luego ningún backtest histórico es admisible como evidencia de promoción.

**Métricas de comparación** (C0 vs C2.0 vs C2.1): retorno acumulado, volatilidad realizada, rotación de composición y de sizing, coste realizado de round-trips, periodo medio de tenencia, solapamiento entre sesiones consecutivas, plazas decididas por desempate (debe ser 0), **distribución sectorial resultante** (dato clave de C2.1), y diferencial contra SPY, QQEW y el universo elegible equiponderado.

---

# Criterios de aceptación

- `crecimiento-v2-c0-sealed.mjs` sin modificar; `methodology_hash` de C0 inalterado.
- Golden `6447af05` en verde; replay `legacy` byte-idéntico sobre las 8 sesiones.
- C2.0 y C2.1 en sombra: cero recomendaciones ejecutables antes de promoción.
- `seleccionar()` sigue siendo pura.
- Suma de control del contrato de factores: 11 + 2 + 2 = 15.
- Todo parámetro nuevo en configuración; ninguno hardcodeado.
- Ninguna dependencia nueva sin aprobación previa.
