---
name: notas-de-venta
description: Registra notas de venta (documento interno NV01 que no va a SUNAT) en Aion Facturación y conviértelas en boleta o factura electrónica con validaciones y vista previa. Úsala cuando el usuario pida una nota de venta, una venta interna sin comprobante todavía, o convertir una nota de venta pendiente en boleta o factura.
---

# Notas de venta y su conversión en boleta o factura

Herramientas del servidor `facturacion`: `company-info`, `search-customers`, `search-products`, `create-sales-note`, `list-sales-notes`, `get-sales-note`, `convert-sales-note`.

Las herramientas de notas de venta solo aparecen si la empresa tiene activo el **módulo de notas de venta**. Si no las ves, dile al usuario que lo solicite en el panel (**Configuración → Licencia y módulos**) y, mientras tanto, ofrece emitir directamente una boleta o factura con la skill `emitir-comprobante`.

## Qué es una nota de venta

Un documento **interno** (serie NV01) que registra la venta **sin valor tributario**: no va a SUNAT. **Nunca la presentes como comprobante de pago.** Úsala solo si el usuario la pide; para una venta formal, emite boleta o factura.

## Registrar una nota de venta

1. Cliente opcional: sin documento queda como CLIENTES VARIOS. Si el cliente tiene **RUC**, regístralo: así podrá convertirse en factura. Al crédito, el cliente debe estar identificado y las cuotas deben sumar el total.
2. Líneas como en una boleta: `product_code` del catálogo (`search-products`) o `description`, `quantity` y `unit_price` con IGV.
3. Genera una `external_reference` única y reutilízala si repites (no se duplica).
4. Llama a `create-sales-note`. No necesita `confirm`, porque no va a SUNAT. Da el número (NV01-5), el total y los enlaces al PDF A4 y al ticket, copiados tal cual.

## Convertir en boleta o factura (con validaciones)

1. **Busca la nota** con `get-sales-note` o `list-sales-notes` (estado *pendiente*). Solo se convierten notas **pendientes**: una convertida ya tiene su comprobante y una anulada no se convierte.
2. **Valida antes de emitir** y explica al usuario lo que encuentres:
   - **Factura (01)**: el cliente necesita **RUC** de 11 dígitos. Si la nota no lo tiene, pide la razón social y el RUC y mándalos en `customer`. La factura conserva el crédito de la nota.
   - **Boleta (03)**: DNI u otro documento. Sin identificar solo hasta el tope de `company-info`; por encima, pide el documento del cliente. **El crédito no vale en boletas**: si la nota era al crédito, pasa a contado o emite factura; dilo al usuario.
   - **Serie**: la del tipo elegido según `company-info` (por defecto, la primera).
   - **Fecha**: hoy por defecto. No uses una fecha anterior sin que el usuario lo pida.
   - Revisa que la empresa pueda emitir (sin requisitos pendientes y con cupo en la licencia).
3. **Vista previa (obligatoria)**: llama a `convert-sales-note` con `"preview": true`. Muestra tipo y serie, cliente, líneas, IGV, total y forma de pago.
4. **Emitir** solo con el visto bueno: con los mismos datos, sin `preview`; en **producción**, además `"confirm": true`.
5. **Resultado**: da el número del comprobante (B001-20), su estado en SUNAT y los enlaces, más el enlace `panel`. La nota pasa a *convertida*. Si SUNAT rechaza, explica el motivo; si no respondió, el sistema reintenta solo.

## Consultar

`list-sales-notes` filtra por pendientes, convertidas o anuladas; `get-sales-note` muestra una con sus enlaces (vencen a los 30 minutos).
