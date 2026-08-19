MIZAN · ACTIVACIÓN FINAL DE GROWTH C1

La entrega de Fase 2B queda aprobada.

HEAD =
c7cd53b

PRODUCTION_SCHEMA =
v31

C1 =
GROWTH_C1_OPERATIONAL_NAV

Estado actual:

C1_STATUS =
APPROVED_PENDING_ACTIVATION

C1_EFFECTIVE_FROM =
NULL

Autorizo activar C1 para la próxima sesión bursátil futura segura.

Entiendo y confirmo que C1 utilizará todo el cash económico reconciliado
disponible, no únicamente las aportaciones futuras.

Esto incluye el cash que ya existe actualmente en la cartera.

==================================================
1. PREVIEW FINAL
==================================================

Antes de activar, ejecutar:

GET /growth-methodology/crecimiento/c1-preview

Verificar y entregar:

- valor de mercado actual;
- cash reconciliado;
- protected cash reserve;
- committed cash;
- investable_nav;
- factor C1/C0;
- sesión de precios;
- próxima sesión aplicable;
- comparación completa por ticker;
- cantidad objetivo C0;
- cantidad objetivo C1;
- acción C0;
- acción C1;
- delta.

Valores aproximados esperados:

market_value =
2.777,10

available_cash =
1.230,00

investable_nav =
4.007,10 aproximadamente

factor =
1,4429 aproximadamente

No exigir coincidencia exacta con esos importes si el precio o el cash
certificado han cambiado. Explicar cualquier diferencia.

Antes de activar, confirmar:

- cash no duplicado;
- settlements no duplicados;
- comisiones contabilizadas;
- committed cash correcto;
- protected reserve correcta;
- holdings reconciliados;
- cero fills Wio pendientes;
- cero partial writes;
- cero movimientos duplicados.

==================================================
2. RESERVA DE CASH
==================================================

Si no existe una reserva de cash configurada:

protected_cash_reserve =
0

No inventar una reserva.

Si existe una reserva canónica previamente configurada:

- respetarla;
- mostrarla en el preview;
- restarla del investable_nav.

No activar si el preview detecta doble conteo de cash o una diferencia
contable no explicada.

==================================================
3. ACTIVACIÓN
==================================================

Si el preview y las invariantes son correctos:

- seguir el procedimiento gap-free de
  backend/CRECIMIENTO-C1-ACTIVATION.md;
- conservar C0 como historia inmutable;
- sellar la versión correspondiente de C0;
- activar C1 con effective_from en la próxima sesión bursátil futura segura;
- no utilizar una fecha retroactiva;
- no modificar la revisión histórica de 2026-07-24;
- no modificar movimientos;
- no modificar holdings;
- no crear una aportación;
- no generar órdenes Wio.

Resultado:

C1_STATUS =
ACTIVE

C1_EFFECTIVE_FROM =
<próxima sesión futura segura>

==================================================
4. PRIMERA REVISIÓN C1
==================================================

La primera revisión C1 debe ser una revisión nueva.

Debe utilizar:

- holdings reales;
- cash reconciliado;
- NAV invertible;
- precios certificados;
- metodología GROWTH_C1_OPERATIONAL_NAV.

Debe generar únicamente:

- MANTENER;
- INCORPORAR;
- AUMENTAR;
- REDUCIR;
- ELIMINAR.

No generar:

- CRP;
- CAPITAL_RESIZING;
- propuesta genérica de resizing.

No ejecutar automáticamente las recomendaciones.

Omar seguirá ejecutándolas manualmente en Wio y registrará posteriormente los
fills reales en Mizan.

==================================================
5. FUTURAS APORTACIONES
==================================================

Después de activar C1, el flujo debe quedar:

1. Omar aporta dinero en Wio.
2. Espera a que esté disponible.
3. Registra la aportación en Mizan.
4. Mizan aumenta cash y NAV sin crear rentabilidad.
5. Mizan crea la solicitud de recálculo.
6. El scheduler genera una nueva revisión C1 en la ventana aplicable.
7. La revisión utiliza el nuevo investable_nav.
8. Omar ejecuta manualmente en Wio.
9. Omar registra los fills reales en Mizan.

==================================================
6. VERIFICACIÓN
==================================================

Después de activar comprobar:

- C1 activa;
- effective_from futura;
- C0 preservada;
- cero cambios retroactivos;
- cero aportaciones creadas por la activación;
- cero movimientos creados por la activación;
- cero órdenes Wio;
- resizing sigue eliminado;
- nuevas crp siguen siendo imposibles;
- Gate 2F sigue inactivo;
- scheduler sano;
- startup catch-up sano;
- Chrome sin errores de Mizan.

==================================================
7. ENTREGA
==================================================

C1_PREVIEW =
PASS/FAIL

MARKET_VALUE =
<importe>

RECONCILED_AVAILABLE_CASH =
<importe>

PROTECTED_CASH_RESERVE =
<importe>

COMMITTED_CASH =
<importe>

INVESTABLE_NAV =
<importe>

C1_C0_FACTOR =
<factor>

CASH_DOUBLE_COUNTING =
0

WIO_FILLS_PENDING =
0

C1_STATUS =
ACTIVE/APPROVED_PENDING_ACTIVATION

C1_EFFECTIVE_FROM =
<fecha/NULL>

C0_PRESERVED =
PASS/FAIL

HISTORICAL_REVIEWS_UNCHANGED =
PASS/FAIL

EXISTING_MOVEMENTS_UNCHANGED =
PASS/FAIL

EXISTING_HOLDINGS_UNCHANGED =
PASS/FAIL

FIRST_C1_REVIEW_STATUS =
SCHEDULED/CREATED/WAITING_FOR_WINDOW

GENERIC_CAPITAL_RESIZING_PROPOSAL_REMOVED =
PASS

NEW_CRP_RECOMMENDATIONS_POSSIBLE =
NO

PRODUCTION_CONTRIBUTIONS_CREATED_BY_ACTIVATION =
0

PRODUCTION_MOVEMENTS_CREATED_BY_ACTIVATION =
0

PRODUCTION_BROKER_ORDERS_SENT =
0

GATE_2F_REMAINS_INACTIVE =
PASS

CHROME =
PASS/FAIL

EXACT_REMAINING_BLOCKERS =
NONE/<lista>

No hacer push.
No crear tag.
Detenerse después de activar y verificar C1.