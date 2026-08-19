DECISIÓN · CIERRE DE PE + TRANSICIÓN CONFORME SEPARADA

La evidencia PIT comenzó:

2026-07-28T19:32:34.276Z

No declarar que comenzó ayer.

El PIT SOURCE_EVIDENCE es independiente del motor de selección.

Mantener separadas:

A. finalización de C2.0;
B. transición prospectiva del motor sealed-tiebreak-compliant.

==================================================
1. PE_HIST · AUDITORÍA DE RECONSTRUCCIÓN PIT
==================================================

Antes de aceptar la referencia anual, verificar si puede construirse una serie
mensual as-known-then mediante componentes históricos.

Por cada mes t:

PE_T =

adjusted_month_end_price_t
/
TTM_diluted_EPS_known_at_t

El TTM EPS conocido en t debe utilizar exclusivamente estados con:

acceptedDate <= month_end_cutoff_t

Auditar:

- profundidad de precios ajustados;
- profundidad de EPS trimestral;
- acceptedDate;
- periodos fiscales;
- restatements;
- splits;
- moneda;
- meses reconstruibles;
- tickers reconstruibles;
- riesgo de look-ahead.

Entregar:

PE_HIST_MONTHLY_PIT_RECONSTRUCTABLE =
YES/NO/PARTIAL

PE_HIST_MONTHLY_VALID_TICKERS =
<número>

PE_HIST_MONTHLY_VALID_MONTHS_MEDIAN =
<número>

PE_HIST_LOOKAHEAD_DETECTED =
YES/NO

No declarar la reconstrucción PIT si se utiliza información publicada después
de cada cutoff mensual.

==================================================
2. DECISIÓN CONDICIONAL PE_HIST
==================================================

Si la reconstrucción mensual PIT es certificable:

PE_HIST_FORMULA_VERSION =
pe_hist_monthly_pit_v1

REFERENCE_WINDOW =
60 meses

MINIMUM_VALID_OBSERVATIONS =
24

Si no es certificable:

No utilizar las observaciones anuales del proveedor como factor canónico.

Definir:

C2_0_CONTINUOUS_FACTOR_COUNT =
10

C2_0_MINIMUM_FACTORS_PRESENT =
8

C2_0_DENOMINATOR =
10

PE_HIST_STATUS =
FORWARD_DIAGNOSTIC_ONLY

PE_HIST_WEIGHT =
0

No mantener un factor estructuralmente neutral para todos como si el
compuesto canónico tuviera once señales.

La futura incorporación de pe_hist será:

C2_0_1

con:

- nuevo methodology_hash;
- preview comparativo;
- shadow paralelo;
- autorización expresa.

En ambos casos, seguir capturando desde ahora:

- current PE as-of;
- precio ajustado;
- TTM diluted EPS;
- acceptedDate;
- period end.

==================================================
3. PE_HIST · CAMBIOS DE RÉGIMEN
==================================================

Si pe_hist entra en el compuesto, reportar diagnósticamente:

- desviación del PE actual frente a su historia;
- observaciones utilizadas;
- dispersión histórica;
- casos extremos.

Marcar:

PE_HISTORY_REGIME_SHIFT_CANDIDATE

cuando la desviación supere el criterio estadístico propuesto y aprobado.

No modificar automáticamente el score por esta bandera.

No asumir que toda desviación histórica extrema representa infravaloración o
sobrevaloración.

==================================================
4. PE_SECT · REFERENCIA APROBADA
==================================================

SECTOR_REFERENCE_UNIVERSE =
FRESH_CERTIFIED_FMP_NASDAQ_ROSTER

La referencia debe ser común para C2.0 y C2.1 dentro de la misma sesión.

Requisitos:

- snapshot staged;
- fresh;
- COMPLETE;
- versionado;
- hash sellado;
- misma taxonomía sectorial FMP;
- acciones ordinarias activas;
- moneda de cotización USD;
- ADR excluidos;
- sector conocido;
- PE positivo y finito.

No aplicar a la referencia:

- sector_regex;
- rev_growth_min;
- cobertura C2;
- top 25;
- capitalización mínima C2;
- liquidez mínima C2.

==================================================
5. MUESTRA SECTORIAL
==================================================

MINIMUM_VALID_SECTOR_PE_OBSERVATIONS =
15

TARGET_VALID_SECTOR_PE_OBSERVATIONS =
30

Si existen menos de 15:

PE_SECT_STATE =
UNDEFINED_SECTOR_REFERENCE

PE_SECT_PERCENTILE =
0.5

No utilizar la mediana general del mercado como fallback.

Persistir:

- sector_reference_universe_version;
- universe_hash;
- cutoff;
- sector;
- miembros;
- member_hash;
- observaciones válidas;
- mediana;
- taxonomía;
- formula_version.

Documentar:

SECTOR_REFERENCE_SCOPE =
NASDAQ_ONLY

NASDAQ_REFERENCE_LIMITATION =
Technology y Healthcare tienen muestras amplias; Energy, Utilities,
Materials, Real Estate y otros sectores pueden tener muestras insuficientes
frente al mercado estadounidense completo.

Una futura ampliación a NYSE exige nueva versión y no puede modificar esta
referencia silenciosamente.

==================================================
6. SURPRISE
==================================================

Mantener la decisión existente:

estimated_eps = 0
→ observación indefinida;
→ excluir del promedio;
→ conservar componentes crudos.

Requerir cuatro observaciones válidas.

SUE permanece como diagnóstico futuro, no como bloqueo ni sustitución de la
fórmula actual.

==================================================
7. DIAGNÓSTICO DE COMPRESIÓN PE
==================================================

Para:

CRWD
INSM
INTC
KHC
MSTR
TTWO
WBD
ZS

entregar:

- pe_hist percentile;
- pe_sect percentile;
- score compuesto;
- rank;
- presencia en top 25;
- distancia a rank 25;
- escenario sin pe_hist;
- escenario sin pe_sect;
- correlación pe_hist/pe_sect.

Declarar:

EFFECT_CLASS =
SCORE_VARIANCE_SHRINKAGE

No cambiar todavía la neutralidad 0.5.

Incluir específicamente TTWO como caso de frontera.

==================================================
8. MOTOR C2
==================================================

Mantener autorizado:

C2_SCORING_ENGINE =
IMPLEMENTED_NOT_SEALED

C2_HYSTERESIS =
IMPLEMENTED_NOT_ACTIVE

No generar la primera sombra real hasta cerrar la decisión condicional de
pe_hist y completar pe_sect.

Si pe_hist es reconstruible:

C2_0_FACTOR_COUNT =
11

Si no es reconstruible:

C2_0_FACTOR_COUNT =
10

No crear una corrida provisional de 9/11.

==================================================
9. TRANSICIÓN SEALED-TIEBREAK-COMPLIANT
==================================================

Preparar en rama y autorización separadas la transición prospectiva de C0/C1
a:

sealed-tiebreak-compliant

No activarlo desde feature/c2-continuous-scoring.

Antes de activar, entregar un dry-run EOD completo usando el snapshot
certificado posterior al cierre del 2026-07-28.

Entregar:

EOD_SNAPSHOT_TIMESTAMP =
<timestamp>

LEGACY_TOP25 =
<lista>

COMPLIANT_TOP25 =
<lista>

FRONTIER_DIFFERENCES =
<lista>

ALPHABETICALLY_DETERMINED_SEATS_LEGACY =
<número exacto>

ALPHABETICALLY_DETERMINED_SEATS_COMPLIANT =
0

PENDING_RECOMMENDATIONS_AFFECTED =
<lista>

ACTIVE_HOLDINGS_AFFECTED =
<lista>

PROPOSED_EFFECTIVE_FROM =
<timestamp de la primera sesión futura aplicable>

No usar estimaciones como:

“unas dos plazas por sesión”

sin entregar el conteo exacto.

==================================================
10. REGLAS DE ACTIVACIÓN CONFORME
==================================================

La activación prospectiva:

- conserva legacy para replay;
- no reescribe sesiones previas;
- no cambia recommendations históricas;
- no reabre PANW;
- no genera órdenes retroactivas;
- no modifica movimientos ni holdings históricos;
- comienza únicamente en effective_from;
- sella selection_engine_version por sesión.

La activación requiere una autorización expresa separada después del dry-run.

Hasta entonces:

COMPLIANT_ENGINE_STATUS =
READY_NOT_ACTIVE

==================================================
11. CHECKPOINT
==================================================

Entregar:

PE_HIST_MONTHLY_PIT_RECONSTRUCTABLE =
YES/NO/PARTIAL

PE_HIST_DECISION =
MONTHLY_PIT_11_FACTOR /
FORWARD_DIAGNOSTIC_10_FACTOR

PE_SECT_REFERENCE_STATUS =
COMPLETE/INCOMPLETE_INPUT

SECTOR_REFERENCE_COUNT =
<número>

SECTORS_BELOW_MINIMUM_15 =
<lista>

C2_0_FACTOR_COUNT =
10/11

C2_0_MINIMUM_FACTORS_PRESENT =
8/9

FACTOR_FORMULA_CONTRACT =
PASS/PENDING

C2_0_METHODOLOGY_HASH =
<hash/NOT_YET>

FIRST_C2_0_SHADOW_OUTPUT =
<estado>

COMPLIANT_EOD_DRY_RUN =
PASS/NOT_RUN

PROPOSED_COMPLIANT_EFFECTIVE_FROM =
<timestamp/NONE>

COMPLIANT_ENGINE_STATUS =
READY_NOT_ACTIVE

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

Detenerse después del checkpoint.

No activar el motor conforme sin autorización separada.
No desplegar C2.