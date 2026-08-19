ACLARACIÓN DE OBJETIVO · UNIVERSO COMPLETO DE CRECIMIENTO

El objetivo final es que Mizan seleccione 25 acciones considerando toda la
relación canónica disponible en:

roster.mjs

Actualmente contiene aproximadamente 1.012 acciones.

No quiero que el objetivo permanente de C2 quede restringido a las 123
acciones de Nasdaq-100 + Dow-22 utilizadas por C0/C1.

Sin embargo, la ampliación debe hacerse de forma secuencial y medible.

==================================================
1. PRESERVACIÓN DE C0/C1
==================================================

C0 y C1 mantienen exactamente su universo actual:

COMPANIES.nasdaq
+
COMPANIES.dow

No modificar:

- construcción de universo C0/C1;
- filtros;
- selección;
- archivos sellados;
- methodology_hash;
- histórico;
- recomendaciones.

La ampliación a roster.mjs pertenece únicamente a una metodología nueva.

==================================================
2. DOS VARIANTES SOMBRA
==================================================

Implementar y medir dos variantes claramente separadas:

C2_0_CONTINUOUS_SCORING_CURATED_UNIVERSE

Universo:

las mismas 123 acciones de C0/C1.

Objetivo:

aislar el efecto de:

- score continuo;
- normalización;
- missing neutral;
- desempate económico;
- histéresis;
- drift band;
- cadencia trimestral.

C2_1_CONTINUOUS_SCORING_FULL_ROSTER

Universo:

toda la relación canónica de roster.mjs que supere los controles de
elegibilidad y calidad.

Objetivo:

medir adicionalmente el efecto de ampliar el universo.

Ambas variantes:

- SHADOW;
- NON_EXECUTABLE;
- sin recomendaciones reales;
- sin cambios en C0/C1;
- con methodology_hash y lineage diferenciados.

==================================================
3. ORDEN DE IMPLEMENTACIÓN
==================================================

El orden obligatorio es:

1. incorporar la especificación normativa;
2. snapshot AS_OBSERVED;
3. archivo PIT forward;
4. motor versionado legacy/compliant;
5. C2.0 sobre las mismas 123 acciones;
6. validar pureza, score, histéresis y auditoría;
7. inventariar roster.mjs;
8. diseñar controles del universo completo;
9. C2.1 sobre el roster completo;
10. ejecutar ambas sombras en paralelo;
11. comparar resultados;
12. no promover ninguna sin evidencia y autorización.

No ampliar primero el universo y añadir después la estabilidad.

==================================================
4. INVENTARIO DEL ROSTER COMPLETO
==================================================

Antes de utilizar roster.mjs, auditarlo read-only.

Entregar:

FULL_ROSTER_RAW_COUNT =
<número>

UNIQUE_SYMBOL_COUNT =
<número>

DUPLICATE_SYMBOLS =
<lista>

ACTIVE_SECURITIES =
<número>

INACTIVE_SECURITIES =
<número>

COMMON_STOCKS =
<número>

ETFS =
<número>

ADRS =
<número>

FUNDS =
<número>

PREFERRED_STOCK =
<número>

OTC =
<número>

DELISTED =
<número>

UNSUPPORTED_EXCHANGES =
<lista>

MISSING_SECTOR =
<número>

MISSING_PRICE =
<número>

MISSING_FUNDAMENTALS =
<número>

MISSING_CURRENCY =
<número>

NON_USD_SECURITIES =
<número>

No asumir que las aproximadamente 1.012 entradas son todas acciones ordinarias
estadounidenses directamente comparables.

==================================================
5. UNIVERSO COMPLETO ELEGIBLE
==================================================

No seleccionar simplemente las primeras filas ni aplicar slice antes de
evaluar todo el roster.

Pipeline esperado:

FULL_ROSTER
→ normalización de símbolos
→ deduplicación
→ instrumentos compatibles
→ mercados admitidos
→ moneda admitida
→ precios certificados
→ fundamentales disponibles
→ política de staleness
→ gates económicos
→ ranking continuo
→ top 25

El ranking debe evaluar todo el universo elegible antes de:

slice(0,25)

No aplicar un límite por orden de archivo, orden alfabético o disponibilidad
accidental.

==================================================
6. POLÍTICA DE ELEGIBILIDAD
==================================================

Antes de implementar C2.1, proponer y esperar aprobación sobre:

- tipos de instrumento permitidos;
- bolsas permitidas;
- moneda;
- capitalización mínima;
- liquidez mínima;
- precio mínimo;
- historial mínimo;
- fundamentales requeridos;
- cobertura mínima de factores;
- política de ADR;
- política de acciones extranjeras;
- política de datos stale;
- corporate actions;
- delistings;
- símbolos sin precio;
- símbolos duplicados.

No reutilizar silenciosamente el gate sectorial de C0/C1.

El universo completo debe permitir comprobar si la concentración tecnológica
es resultado del modelo o de una restricción explícita.

Si se conserva algún gate sectorial en C2.1, debe quedar claramente
configurado y justificado.

==================================================
7. PIT DEL UNIVERSO COMPLETO
==================================================

El archivo PIT debe poder capturar el roster completo, no solo las 123
acciones actuales.

Por sesión registrar:

- roster total;
- miembros considerados;
- miembros excluidos;
- motivo de exclusión;
- raw values;
- fechas económicas;
- acceptedDate;
- fetchedAt;
- precio as-of;
- missing;
- staleness;
- score;
- rank;
- methodology hash;
- input hash;
- content hash.

No consultar aproximadamente 1.012 símbolos de forma que:

- bloquee el scheduler;
- supere rate limits;
- produzca un universo parcial silencioso;
- mezcle sesiones o cutoffs;
- degrade C0/C1.

Diseñar fetch, staging, validación y sellado antes de activar C2.1.

==================================================
8. COMPARACIÓN C2.0 VS C2.1
==================================================

Durante la sombra medir para ambas:

- número bruto de miembros;
- número elegible;
- cobertura;
- top 25;
- score de la frontera;
- dispersión de scores;
- plazas decididas por desempate;
- composición;
- sectores;
- concentración;
- turnover potencial;
- overlap entre sesiones;
- retorno teórico TWR;
- volatilidad;
- drawdown;
- costes teóricos;
- diferencial frente a SPY;
- diferencial frente a QQEW;
- diferencial frente a su universo elegible equiponderado.

Entregar además:

C2_0_VS_C2_1_TOP25_OVERLAP =
<porcentaje>

C2_1_NEW_NAMES_NOT_IN_CURATED_123 =
<lista>

C2_1_SECTOR_DISTRIBUTION =
<lista>

C2_1_ELIGIBLE_COUNT =
<número>

No asumir que un universo mayor produce necesariamente mejor selección.

==================================================
9. PREVIEW ACTUAL
==================================================

El preview:

ABNB fuera
PANW dentro

es válido únicamente como diagnóstico del desempate sellado dentro del
universo legacy.

No presentarlo como preview de C2 sobre el roster completo.

Declarar:

LEGACY_UNIVERSE_TIEBREAK_PREVIEW =
COMPLETE

FULL_ROSTER_C2_PREVIEW =
NOT_YET_AVAILABLE

No utilizar ese preview para reabrir PANW ni para ejecutar operaciones.

==================================================
10. CHECKPOINT
==================================================

Confirmar:

FINAL_TARGET_UNIVERSE =
FULL_CANONICAL_ROSTER

C0_C1_UNIVERSE_UNCHANGED =
PASS/FAIL

C2_0_UNIVERSE =
CURATED_123

C2_1_UNIVERSE =
FULL_ROSTER

FULL_ROSTER_RAW_COUNT =
<número>

FULL_ROSTER_AUDIT =
PASS/FAIL

FULL_ROSTER_ELIGIBILITY_POLICY =
PENDING_APPROVAL/APPROVED

C2_0_STATUS =
PLANNED/SHADOW

C2_1_STATUS =
PLANNED/SHADOW

C2_0_AND_C2_1_HASHES_DISTINCT =
PASS/FAIL

FULL_ROSTER_SELECTION_EXECUTABLE =
NO

PRODUCTION_RECOMMENDATIONS_CREATED =
0

PRODUCTION_WRITES =
0

Detenerse antes de implementar C2.1 hasta entregar el inventario y la política
de elegibilidad del roster completo.