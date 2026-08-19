MIZAN · APROBACIÓN DE POLÍTICA DE UNIVERSO C2
Y AUTORIZACIÓN DE CONSTRUCCIÓN EN RAMA

El contenido íntegro de:

spec-mizan-c2-seleccion.md

precede a esta orden y es normativo.

El checkpoint read-only del roster queda aprobado.

BASELINE

BRANCH =
feature/c2-continuous-scoring

PRODUCTION_HEAD =
1d67c54

PRODUCTION_SCHEMA =
v31

FULL_ROSTER_CURRENT_COUNT =
1060

==================================================
1. OBJETIVO FINAL
==================================================

El universo objetivo final de Crecimiento no queda limitado a las 123
acciones de C0/C1.

Construir dos variantes sombra independientes:

C2_0_CONTINUOUS_SCORING_CURATED_UNIVERSE

Universo:
123 acciones legacy.

Finalidad:
aislar el efecto del nuevo score, normalización, histéresis, desempate y drift
band.

C2_1_CONTINUOUS_SCORING_FULL_ROSTER

Universo:
roster canónico completo, actualmente de 1.060 entradas, después de los
controles de elegibilidad aprobados.

Finalidad:
medir adicionalmente el efecto de ampliar el universo.

Ambas variantes deben tener:

- configuración sellada independiente;
- methodology_hash distinto;
- lineage distinto;
- status SHADOW;
- executable FALSE;
- cero recomendaciones reales.

==================================================
2. PRESERVACIÓN DE C0/C1
==================================================

No modificar:

- universo de C0/C1;
- COMPANIES.nasdaq;
- COMPANIES.dow;
- gates;
- score;
- top 25;
- archivos sellados;
- methodology_hash;
- selección activa;
- recomendaciones;
- histórico.

C0_C1_UNIVERSE =
CURATED_123

C0_C1_UNIVERSE_CHANGED =
NO

==================================================
3. POLÍTICA DE INSTRUMENTOS
==================================================

C2.1 canónica admite inicialmente:

INSTRUMENT_TYPE =
COMMON_STOCK_ONLY

Excluir:

- ETF;
- fondos;
- warrants;
- rights;
- units;
- preferred stock;
- instrumentos no clasificables;
- instrumentos inactivos;
- símbolos no negociables.

No confiar únicamente en heurísticas del símbolo.

Realizar enriquecimiento de tipo de instrumento en staging.

==================================================
4. ADR Y EMISORES EXTRANJEROS
==================================================

Política inicial canónica:

ADR_POLICY =
EXCLUDE_FROM_CANONICAL_C2_1

FOREIGN_ISSUER_POLICY =
EXCLUDE_IF_NOT_NORMALIZED

No eliminar los ADR del inventario ni ocultarlos.

Registrarlos como:

EXCLUDED_ADR_OR_FOREIGN_ISSUER

y entregar una variante diagnóstica que indique:

- símbolo;
- país;
- moneda de reporte;
- ratio ADR;
- fuente de fundamentales;
- disponibilidad;
- efecto potencial de incluirlo.

No crear todavía una variante ejecutable con ADR.

Su incorporación posterior exige normalización de:

- ratio ADR;
- moneda;
- factores monetarios;
- source period;
- disclosure cadence;
- costes específicos si aplican.

==================================================
5. BOLSA Y MONEDA
==================================================

C2.1 inicial:

EXCHANGE =
NASDAQ

TRADING_CURRENCY =
USD

No ampliar todavía a NYSE.

NYSE será una ampliación posterior separada, para no mezclar:

- cambio de motor;
- ampliación de roster;
- ampliación de bolsa.

Registrar la moneda de reporte de fundamentales.

Si un factor monetario no puede normalizarse de manera reproducible:

- marcarlo missing;
- aplicar política de cobertura;
- no mezclar importes de monedas diferentes.

==================================================
6. CAPITALIZACIÓN
==================================================

Configurar para la variante canónica:

MINIMUM_MARKET_CAP_USD =
300000000

No utilizar:

2000000000

como único corte inicial, porque eliminaría una parte excesiva del roster y
volvería a restringir artificialmente el universo.

Ejecutar además análisis de sensibilidad read-only para:

MINIMUM_MARKET_CAP_USD =
1000000000

y:

MINIMUM_MARKET_CAP_USD =
2000000000

Entregar para cada umbral:

- miembros elegibles;
- cobertura;
- sectores;
- top 25;
- overlap;
- score de frontera;
- turnover sombra;
- exclusiones.

No optimizar el umbral usando retorno histórico contaminado.

==================================================
7. LIQUIDEZ
==================================================

No utilizar únicamente el volumen almacenado en el roster.

Calcular:

MEDIAN_20_SESSION_DOLLAR_VOLUME =
median(close × volume)

sobre sesiones certificadas.

Umbral canónico inicial:

MINIMUM_MEDIAN_20D_DOLLAR_VOLUME_USD =
1000000

Ejecutar análisis de sensibilidad para:

5000000

y:

10000000

No seleccionar un ticker si faltan datos suficientes para calcular la
liquidez de forma certificable.

Registrar:

- ventana;
- sesiones válidas;
- volumen;
- precio;
- dollar volume;
- fuente;
- as-of.

==================================================
8. PRECIO E HISTORIA
==================================================

MINIMUM_PRICE_USD =
5.00

MINIMUM_LISTING_HISTORY_YEARS =
10

El historial de diez años del roster debe verificarse contra la fecha de la
sesión, no asumirse permanentemente por el build del 2026-07-06.

Excluir:

- símbolos sin precio certificado;
- símbolos suspendidos;
- símbolos inactivos;
- símbolos sin una sesión comparable;
- delistings confirmados.

==================================================
9. FUNDAMENTALES Y POINT-IN-TIME
==================================================

Requerir:

- filing disponible;
- acceptedDate;
- period end;
- fetchedAt;
- fuente;
- cutoff;
- valores crudos;
- staleness por factor.

Regla:

acceptedDate <= session_cutoff

No utilizar un filing publicado después del cutoff.

No inferir la fecha económica del dato desde el mtime del archivo.

La caché productiva write-once no debe refrescarse en esta fase.

El enriquecimiento se realiza en:

- staging;
- artefactos nuevos;
- snapshots PIT;
- sin sobrescribir fund_*.json productivo.

==================================================
10. COBERTURA DE FACTORES
==================================================

No aprobar todavía:

minimum_checks =
10 de 15

porque sigue sin estar cerrado el contrato:

11 factores
frente a
15 checks.

Primero entregar:

ACTIVE_BINARY_CHECKS =
<lista>

CONTINUOUS_RANKING_FACTORS =
<lista>

ELIGIBILITY_GATES =
<lista>

DISPLAY_ONLY_CHECKS =
<lista>

RISK_ONLY_CHECKS =
<lista>

Después aplicar:

MINIMUM_CONTINUOUS_FACTOR_COVERAGE =
80_PERCENT

La cobertura se calcula sobre los factores continuos aprobados, no sobre todos
los checks de display o gates.

Ejemplo:

approved_continuous_factors =
10

minimum_required =
8

Los factores ausentes después de superar el mínimo reciben:

neutral_percentile =
0.5

Si no se alcanza el mínimo:

ELIGIBILITY_STATUS =
INSUFFICIENT_FACTOR_COVERAGE

==================================================
11. GATE SECTORIAL
==================================================

C2.1 no tendrá gate sectorial:

SECTOR_GATE =
NONE

No reutilizar:

technolog|consumer cyclical|discretionary

El sector se utiliza para:

- análisis;
- exposición;
- concentración;
- comparaciones;
- límites futuros si se aprueban;

pero no para excluir candidatos en la primera sombra C2.1.

El objetivo es medir si la concentración surge del ranking o de una
restricción impuesta.

No añadir todavía cupos sectoriales.

==================================================
12. DATOS AUSENTES Y STALE
==================================================

Distinguir:

STRUCTURAL_MISSING

Ejemplos:

- instrumento incompatible;
- precio inexistente;
- moneda no normalizable;
- filing requerido inexistente.

Resultado:

EXCLUDE

FACTOR_LEVEL_MISSING

Existe ticker elegible, pero falta un factor individual.

Resultado:

- neutral 0.5;
- registrar reason;
- sujeto al mínimo de cobertura.

STALE

El dato existe, pero incumple la política de frescura de su clase.

Resultado:

- registrar staleness;
- no tratarlo silenciosamente como fresco;
- aplicar política específica del factor.

No utilizar un TTL único para:

- precios;
- filings;
- estimaciones;
- referencias estáticas.

==================================================
13. ROSTER Y ENRIQUECIMIENTO
==================================================

El roster del 2026-07-06 tiene 22 días de antigüedad.

Antes de ejecutar C2.1, construir un pipeline de staging:

roster snapshot
→ enrichment
→ validation
→ exclusions
→ PIT seal
→ shadow compute

No sobrescribir:

export/roster-cache.json

durante la construcción.

No aceptar un universo parcial silencioso.

Cada ejecución debe entregar:

RAW_COUNT =
<número>

ENRICHED_COUNT =
<número>

FAILED_ENRICHMENT_COUNT =
<número>

EXCLUDED_COUNT =
<número>

ELIGIBLE_COUNT =
<número>

FAILURE_REASONS =
<lista>

Si se interrumpe por rate limit o error:

- no sellar como universo completo;
- no calcular top 25 canónico;
- marcar INCOMPLETE_INPUT;
- reintentar idempotentemente.

==================================================
14. AS-OBSERVED Y PIT
==================================================

Autorizo construir en esta rama, sin Producción:

AS_OBSERVED_INPUT_SNAPSHOT

y:

PIT_FORWARD_ARCHIVE

El primer snapshot debe preservar lo que consume actualmente C0/C1 antes de
cualquier refresh.

Después, el PIT de C2 debe soportar:

- C2.0;
- C2.1;
- roster completo;
- motivos de exclusión;
- factor inputs;
- fechas;
- hashes;
- lineage.

No presentar AS_OBSERVED como histórico PIT retroactivo.

==================================================
15. AUTORIZACIÓN DE CONSTRUCCIÓN
==================================================

Confirmo:

“preparar/iniciar”

significa:

- construir código en feature/c2-continuous-scoring;
- crear tests;
- crear previews;
- generar artefactos de LAB/staging;
- realizar cálculos sombra;
- cero deploy productivo;
- cero activación;
- cero recomendaciones ejecutables;
- cero escrituras económicas.

Autorizado en esta rama:

- documento normativo;
- AS_OBSERVED;
- archivo PIT;
- C2.0;
- C2.1;
- score continuo;
- histéresis pura;
- shadow portfolio;
- comparadores.

No autorizado:

- modificar C0/C1;
- activar el motor conforme en Producción;
- refrescar la caché fundamental productiva;
- cambiar active selection;
- crear recomendaciones reales;
- enviar órdenes a Wio.

==================================================
16. DESEMPATE SELLADO
==================================================

El preview:

ABNB fuera
PANW dentro

queda clasificado exclusivamente como:

LEGACY_UNIVERSE_TIEBREAK_DIAGNOSTIC

No representa C2.1.

No reabre PANW.

No genera operaciones.

La infraestructura legacy/compliant puede prepararse y probarse, pero
cualquier activación prospectiva para C0/C1 debe realizarse en una rama y
autorización separadas.

No mezclar esa activación con C2.

==================================================
17. CHECKPOINT ANTES DE C2.1 COMPLETA
==================================================

Entregar:

NORMATIVE_SPEC_PRESENT =
PASS/FAIL

FULL_ROSTER_RAW_COUNT =
1060

FULL_ROSTER_BUILD_DATE =
2026-07-06

INSTRUMENT_ENRICHMENT =
PASS/FAIL

COMMON_STOCK_COUNT =
<número>

ADR_EXCLUDED_COUNT =
<número>

MARKET_CAP_300M_ELIGIBLE =
<número>

MARKET_CAP_SENSITIVITY =
<resumen>

LIQUIDITY_1M_ELIGIBLE =
<número>

LIQUIDITY_SENSITIVITY =
<resumen>

PRICE_5_USD_ELIGIBLE =
<número>

CONTINUOUS_FACTOR_CONTRACT =
PASS/FAIL

MINIMUM_FACTOR_COVERAGE =
80_PERCENT

SECTOR_GATE =
NONE

C2_0_METHODOLOGY_HASH =
<hash>

C2_1_METHODOLOGY_HASH =
<hash>

C2_0_STATUS =
SHADOW

C2_1_STATUS =
SHADOW/INCOMPLETE_INPUT

AS_OBSERVED_SNAPSHOT =
PASS/FAIL

PIT_FORWARD_ARCHIVE =
PASS/FAIL

C0_C1_UNCHANGED =
PASS/FAIL

PRODUCTION_DEPLOY =
NO

PRODUCTION_WRITES =
0

PRODUCTION_RECOMMENDATIONS =
0

BROKER_ORDERS =
0

EXACT_DECISIONS_REQUIRED_FROM_OMAR =
<lista/NONE>

Detenerse en este checkpoint.

No desplegar.
No activar C2.