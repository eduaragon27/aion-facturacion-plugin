---
name: emitir-comprobante
description: Emite facturas, boletas, notas de crédito y notas de débito electrónicas ante SUNAT con Aion Facturación, con vista previa y validaciones antes de enviar. Úsala cuando el usuario pida facturar, emitir una boleta o factura, hacer una nota de crédito o débito, anular o corregir una venta con una nota, o registrar un cliente o producto para facturar.
---

# Emitir un comprobante electrónico

Herramientas del servidor `facturacion`: `company-info`, `search-customers`, `save-customer`, `search-products`, `save-product`, `exchange-rate`, `issue-document`, `get-document`, `retry-sunat`, `email-document`.

## Paso a paso

1. **Empresa**: llama a `company-info` si aún no lo hiciste en la conversación. Anota el entorno (beta o producción), las series de cada tipo y el tope de la boleta sin identificar. Si falta algo para emitir, dilo y detente.
2. **Tipo de comprobante**:
   - **Factura (01)**: el cliente debe tener **RUC** (`document_type` "6", 11 dígitos). Serie que empieza con F.
   - **Boleta (03)**: DNI, carné de extranjería, pasaporte o sin documento. Serie que empieza con B. Sin identificar: `{"document_type": "0", "document_number": "-", "name": "CLIENTES VARIOS"}`, solo hasta el tope que indica `company-info`; por encima hay que identificar al cliente.
   - **Nota de crédito (07) o débito (08)**: corrige un comprobante **aceptado**. Usa la serie de notas del mismo tipo (FC01 para facturas, BC01 para boletas) y `affected_document` con su número (F001-15).
   - Si el usuario no lo dice: con RUC, factura; sin RUC, boleta. Pregunta si hay duda.
3. **Cliente**: búscalo con `search-customers`. Si no existe, pide los datos que falten (nunca los inventes) y créalo con `save-customer`, o mándalo dentro de `customer`.
4. **Productos**: búscalos con `search-products` y usa su `product_code`; toman descripción, precio y afectación del catálogo. Para algo que no está en el catálogo, manda `description`, `quantity`, `unit_price` (con IGV) y `unit_code` (NIU bienes, ZZ servicios). Si el producto se venderá otra vez, ofrece guardarlo con `save-product`.
5. **Forma de pago** (facturas): contado por defecto. Al crédito, cuotas con monto y fecha que sumen el total y venzan después de la emisión.
6. **Referencia única**: genera una `external_reference` única por venta (por ejemplo `chat-AAAAMMDD-HHMM-cliente`) y **reutilízala** si repites la llamada. Así un reintento devuelve el mismo comprobante (`already_issued`) y nunca duplica.
7. **Vista previa (obligatoria)**: llama a `issue-document` con `"preview": true`. Muestra al usuario un resumen claro: tipo y serie, cliente, cada línea con cantidad y precio, subtotal, IGV y total, forma de pago y entorno. Pregunta si lo emites.
8. **Emitir** solo con el visto bueno del usuario, con **los mismos datos**:
   - En **beta**: sin `preview`.
   - En **producción**: sin `preview` y con `"confirm": true`. Recuerda al usuario que el comprobante es real ante SUNAT.
9. **Resultado**: da el número (F001-15), el estado en SUNAT y los enlaces `links` (PDF, XML, CDR) copiados tal cual, más el enlace `panel`.
   - **Aceptado u observado**: listo. Ofrece enviarlo por correo con `email-document`.
   - **Rechazado**: explica el motivo de SUNAT; hay que corregir y emitir uno nuevo.
   - **Error o pendiente** (SUNAT no respondió): el sistema lo reintenta solo; también puedes usar `retry-sunat` con su `document_id`. No lo emitas de nuevo con otra referencia.

## Notas de crédito y débito

- Motivos de crédito más usados (catálogo 09): `01` anulación de la operación, `06` devolución total, `07` devolución por ítem, `13` ajuste de cuotas o fechas de pago. En 01, 02 y 06 las líneas se copian solas del original; en 13 no se envían líneas, sino las cuotas nuevas.
- Pide siempre el motivo (`reason`) en palabras del usuario.
- Antes, revisa el original con `get-document`: debe estar aceptado y su saldo por acreditar alcanza para la nota.
- Si el usuario quiere **anular** un comprobante reciente sin nota, usa la skill `consultar-ventas` (baja con `void-document`).

## Errores frecuentes

- Los errores llegan en español con el campo exacto (`- customer.document_number: …`). Corrige ese dato y repite la vista previa.
- Dólares: `currency` "USD" con el precio de cada línea; el tipo de cambio lo pone el servicio (consúltalo con `exchange-rate` si el usuario lo pide).
- Inventario por series: si el producto lo exige, cada línea lleva `serial_numbers` (una serie por unidad).
- Un comprobante aceptado **no se edita**: se corrige con nota o se da de baja.

No calcules IGV ni totales: los calcula el servicio y la vista previa los muestra.
