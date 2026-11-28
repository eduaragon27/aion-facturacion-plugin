---
name: cotizaciones
description: Prepara cotizaciones (propuestas COT1 que no van a SUNAT ni consumen cupo) en Aion Facturación, envíalas al cliente con un enlace para aceptarlas o rechazarlas y conviértelas en boleta, factura o nota de venta. Úsala cuando el usuario pida una cotización, un presupuesto o una proforma, quiera mandarla a un cliente o convertir una cotización aceptada.
---

# Cotizaciones

Herramientas del servidor `facturacion`: `company-info`, `search-customers`, `search-products`, `create-quote`, `search-quotes`, `get-quote`, `email-quote`, `convert-quote`.

Una cotización es una **propuesta comercial**: no es comprobante de pago, no va a SUNAT y no consume el cupo del plan (viene en todos los planes). El cliente la recibe con un enlace donde la ve, descarga el PDF y la **acepta o rechaza** hasta la fecha de validez. Recién al convertirla se emite el comprobante, con las reglas y el cupo de siempre.

`create-quote` necesita el nivel *Consultar y registrar*; `email-quote` y `convert-quote`, el nivel *También emitir*. Si no ves una herramienta, explícale al usuario que debe autorizar ese nivel (en Claude Code, `/mcp` → `facturacion`).

## 1. Antes de cotizar

1. Llama a `company-info` para conocer la empresa, el entorno y sus series.
2. Busca al cliente con `search-customers`. La cotización necesita al menos el **nombre o razón social**; con **RUC** podrá convertirse en factura. Pregunta si va dirigida a una persona («Atención a») y su correo, si la enviarás por correo.
3. Busca los productos con `search-products`. Con `product_code` la línea toma la descripción, el precio y la imagen del catálogo. Para un producto o servicio nuevo, usa `description`, `quantity` y `price`. **No inventes precios**: si no están en el catálogo, pregúntalos.

## 2. Pregunta siempre

- **Modo de IGV** (`igv_mode`):
  - `included`: los precios ya incluyen IGV (lo habitual con consumidores).
  - `added`: precios sin IGV; el IGV se desglosa al final (lo habitual entre empresas).
  - `none`: sin IGV (operaciones no gravadas o una cotización referencial). Al convertirla, el comprobante agrega el IGV de las líneas gravadas: avísalo.
  El `price` de cada línea se interpreta según ese modo.
- **Validez**: una fecha (`valid_until`) o días (`validity_days`). Si el usuario no dice nada, se usa la de la empresa; dile cuál quedó.
- Si quiere cambiar las condiciones (forma de pago, plazo de entrega, garantía, observaciones), los datos bancarios, la firma o la plantilla (`classic`, `modern` o `minimal`). Si no, quedan los valores por defecto de la empresa.

## 3. Vista previa obligatoria

1. Llama a `create-quote` con `"preview": true`. No registra nada.
2. Muestra al usuario: cliente, líneas con cantidad y precio, subtotal, IGV (si corresponde), total, validez y modo de IGV.
3. Solo con su visto bueno, repite la llamada **con los mismos datos y sin `preview`**. Usa una `external_reference` única y reutilízala si reintentas: no se duplica.
4. Da el número (COT1-5), el total y los enlaces **copiados tal cual**: `links.pdf` (vence a los 30 minutos), `links.public` (el enlace para el cliente, no vence) y `links.panel`.

Para una línea con imagen propia, manda `image_url` con una dirección **https** pública de una imagen JPEG, PNG o WebP (hasta 5 MB). Si el servidor la rechaza, explícale el motivo al usuario y ofrece seguir sin imagen.

## 4. Enviar al cliente

Ofrece enviarla con `email-quote` (por defecto al correo del cliente; confirma la dirección antes). Otra opción es que el usuario comparta `links.public` por mensaje. La cotización pasa de *borrador* a *enviada*. Una cotización vencida no se envía: hay que editar su validez en el panel.

## 5. Seguimiento y conversión

1. Consulta el estado con `search-quotes` (filtros `draft`, `sent`, `accepted`, `rejected`, `expired`, `converted`) o `get-quote`, que trae el historial y el comentario del cliente.
2. Cuando el cliente la **acepte** (o siga enviada y vigente), ofrece convertirla con `convert-quote`:
   - En **boleta o factura**: por defecto factura si el cliente tiene RUC; si no, boleta, al contado. Aplican las validaciones de la skill `emitir-comprobante` (la factura exige RUC; la boleta, identificar al cliente sobre el tope). Primero con `"preview": true`; muestra la vista previa y emite solo con el visto bueno. En **producción**, además `"confirm": true`.
   - En **nota de venta**: `"target": "sales_note"`, solo si la empresa tiene ese módulo (skill `notas-de-venta`).
3. Repetir la conversión devuelve el mismo comprobante: no se duplica. Da el número del comprobante, su estado en SUNAT y sus enlaces.

## Reglas

- Nunca presentes una cotización como comprobante de pago.
- No calcules totales ni IGV tú: los calcula el servicio.
- Una cotización aceptada o convertida ya no se edita; para otra versión, el usuario la duplica en el panel.
