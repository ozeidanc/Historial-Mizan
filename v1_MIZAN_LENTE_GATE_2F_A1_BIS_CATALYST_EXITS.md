MIZAN · LA LENTE
GATE 2F-A1-BIS · FUNDACIÓN TRANSACCIONAL DEL HARD CAP
COMPRAS + ALERTAS DE CATALIZADOR + REDUCCIONES + SALIDAS
RESERVA DE CASH Y CANTIDAD · LAB ÚNICAMENTE

AMPLIACIÓN OBLIGATORIA DEL ALCANCE

Además de las allocation requests para incorporar nuevas tesis, Gate 2F debe
modelar solicitudes de acción sobre posiciones existentes cuando Mizan
detecte:

- catalizador ya no activo;
- catalizador revertido;
- catalizador completado;
- otra condición de revisión de salida.

La alerta es exclusivamente informativa.

Mizan debe ofrecer:

1. MANTENER;
2. REDUCIR POSICIÓN;
3. VENDER POSICIÓN COMPLETA.

Mizan no puede vender, reducir ni cerrar automáticamente una posición como
consecuencia del estado del catalizador.

==================================================
1. PRINCIPIO DE DECISIÓN DEL PROPIETARIO
==================================================

La detección confirmada en dos escaneos consecutivos:

- crea o actualiza una alerta;
- no crea una transacción;
- no modifica la composición;
- no cierra el membership;
- no aumenta Paper Cash;
- no cambia Paper NAV;
- no cambia Paper Unit Value;
- no modifica el Track.

La decisión pertenece exclusivamente a Omar.

Estados de decisión posibles:

- ACTION_REQUIRED;
- HOLD_ACKNOWLEDGED;
- REDUCTION_DRAFT;
- EXIT_DRAFT;
- AWAITING_CONFIRMATION;
- RESERVED_FOR_EXECUTION;
- PENDING_MARKET_CLOSE;
- PENDING_PRICE;
- EXECUTED;
- CANCELLED;
- REJECTED;
- FAILED;
- SUPERSEDED.

Gate 2F-A1-BIS solo llega hasta:

- HOLD_ACKNOWLEDGED;
- REDUCTION_DRAFT;
- EXIT_DRAFT;
- AWAITING_CONFIRMATION;
- RESERVED_FOR_EXECUTION;
- CANCELLED;
- REJECTED;
- FAILED;
- SUPERSEDED.

No ejecuta todavía ventas.

==================================================
2. ALERTAS ACTUALES
==================================================

El sistema muestra actualmente alertas como:

CATALYST_NO_LONGER_ACTIVE:

- ONB;
- ATRC;
- ISRG;
- HCSG;
- WSBC;
- WTFC.

CATALYST_REVERTED:

- PEGA.

Este inventario es una entrada observada, no una autorización para operar.

Antes de ofrecer una acción económica, reconciliar cada ticker con:

- paper_global_position;
- membership episode abierto;
- cartera papel actual;
- composición económica vigente;
- cantidad económica canónica;
- última sesión certificada.

No inferir una cartera a partir de la categoría C1, C2, C3, C4 o C6.

La categoría del catalizador y la cartera papel son dimensiones diferentes.

==================================================
3. ELEGIBILIDAD DE LA ACCIÓN
==================================================

Mostrar REDUCIR o VENDER únicamente cuando exista:

- global position identificada;
- membership económico abierto;
- cantidad económica canónica > 0;
- cartera papel activa;
- último snapshot canónico;
- lineage resuelto.

Si el ticker:

- no pertenece a ninguna cartera papel;
- solo tiene un membership de auditoría sin exposición;
- ya fue vendido;
- tiene cantidad cero;
- tiene una salida pendiente por toda la cantidad;

mostrar únicamente información, no controles de venta.

Si una posición existe en más de una cartera:

- listar cada cartera;
- mostrar cantidad y valor por cartera;
- exigir seleccionar una;
- no ejecutar una acción global implícita.

==================================================
4. OPCIÓN MANTENER
==================================================

MANTENER significa:

- conservar posición y cantidad;
- conservar membership;
- no crear transacción;
- no cambiar Paper Cash;
- no cambiar el Track.

Registrar una decisión append-only:

decision_type =
HOLD_ACKNOWLEDGED

Campos:

- global_position_id;
- membership_episode_id;
- paper_portfolio_id;
- catalyst reference;
- catalyst status observado;
- scan confirmation reference;
- decided_by;
- decided_at_utc;
- rationale opcional;
- review_again_at nullable;
- content_hash.

La decisión MANTENER no debe:

- reactivar el catalizador;
- modificar la tesis;
- ocultar permanentemente futuras alertas;
- impedir que una nueva evidencia vuelva a abrir la revisión.

==================================================
5. OPCIÓN REDUCIR POSICIÓN
==================================================

REDUCIR debe permitir especificar de forma segura:

- porcentaje de la cantidad económica;
- o cantidad exacta.

Para la primera versión no utilizar un importe monetario fijo como instrucción
canónica, porque el precio efectivo del cierre todavía no es conocido.

La UI puede mostrar un notional estimado usando el último cierre certificado,
pero debe etiquetarlo:

ESTIMACIÓN, NO PRECIO DE EJECUCIÓN.

Campos de la solicitud:

- global_position_id;
- membership_episode_id;
- paper_portfolio_id;
- action_type = PARTIAL_EXIT;
- requested_reduction_basis =
  PERCENTAGE o QUANTITY;
- requested_reduction_value;
- current_canonical_quantity;
- proposed_reserved_quantity;
- estimated_notional;
- estimate_session;
- catalyst reference;
- catalyst status;
- requested_by;
- requested_at_utc;
- status;
- idempotency_key;
- content_hash.

Debe cumplirse:

0 < reserved_quantity < current_available_quantity

Una reducción parcial:

- mantiene abierto el membership;
- reduce la cantidad al ejecutarse;
- aumenta Paper Cash;
- no emite ni retira Paper Units;
- no cambia Paper Unit