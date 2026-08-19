CORRECCIÓN OBLIGATORIA DEL CASO PANW

La operación real que debe registrarse es:

ticker = PANW
executed_quantity = 0.33422822
executed_price = 331.51
executed_gross_amount = 110.80
commission = 0.52
total_cash_debit = 111.32
execution_date = 2026-07-24

Semántica obligatoria:

- 110.80 es el IMPORTE BRUTO DE LA COMPRA;
- 0.52 es la COMISIÓN, registrada separadamente;
- 111.32 es únicamente el EFECTO TOTAL SOBRE CASH;
- no guardar 111.32 como importe comprado;
- no sustituir la ejecución por la recomendación de 110.85;
- no sustituir la cantidad real por 0.342213.

El formulario debe aceptar y registrar:

cantidad × precio = importe bruto

y persistir separadamente:

gross_amount
commission
total_cash_effect

Para BROKER_EXECUTED_FILL:

- CASH_OVERSPEND_BLOCKED no puede impedir el registro;
- registrar el fill real;
- actualizar el Libro Mayor;
- actualizar el holding;
- aplicar el efecto total de cash;
- si el cash no reconcilia, marcar CASH_RECONCILIATION_REQUIRED;
- conservar movement_id y trazabilidad hacia la recomendación.

Resultado obligatorio:

PANW_LEDGER_GROSS_AMOUNT = 110.80
PANW_LEDGER_COMMISSION = 0.52
PANW_LEDGER_TOTAL_CASH_DEBIT = 111.32
PANW_HOLDING_QUANTITY_ADDED = 0.33422822
PANW_EXECUTION_RECORDED = PASS
PANW_DUPLICATE_MOVEMENTS = 0