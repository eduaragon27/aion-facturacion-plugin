---
name: consultar-ventas
description: Consulta las ventas y comprobantes emitidos en Aion Facturación, reenvía comprobantes por correo, reintenta los que no llegaron a SUNAT y da de baja comprobantes aceptados. Úsala cuando el usuario pregunte cuánto vendió, busque una factura o boleta, quiera el PDF o XML, pida enviar un comprobante al cliente, revisar su estado en SUNAT o anular un comprobante.
---

# Consultar ventas y gestionar comprobantes

Herramientas del servidor `facturacion`: `sales-report`, `list-documents`, `get-document`, `email-document`, `retry-sunat`, `void-document`, `exchange-rate`.

## Reporte de ventas

- `sales-report` con `from` y `to` (AAAA-MM-DD, ambos obligatorios). Da las ventas en soles **solo de comprobantes aceptados**, por tipo y con los principales clientes.
- Presenta el total, el desglose por tipo y los 3 a 5 principales clientes. Si el usuario quiere una tabla o gráfico, arma uno con esos datos; no inventes cifras.
- Aclara el entorno: en beta son ventas de prueba.

## Buscar comprobantes

- `list-documents` filtra por tipo, estado, fechas o texto (cliente, número, referencia).
- `get-document` (por `id`, `full_number` como F001-15 o `external_reference`) trae líneas, estado en SUNAT, notas asociadas, saldo por acreditar y los enlaces.
- Los enlaces de `links` vencen a los 30 minutos: cópialos exactamente; si vencieron, vuelve a llamar a `get-document`. Da también el enlace `panel`.

## Estados en SUNAT

| Estado | Qué significa | Qué hacer |
| --- | --- | --- |
| Aceptado | Válido | Nada |
| Observado | Válido con observaciones | Leer la observación |
| Rechazado | No válido | Corregir y emitir uno nuevo |
| Pendiente o error | SUNAT no respondió | El sistema reintenta solo; o usar `retry-sunat` con `document_id` |
| Dado de baja | Anulado ante SUNAT | Definitivo |

## Enviar por correo

`email-document` envía el PDF y el XML de un comprobante **aceptado**. Sin `email` usa el correo del cliente; si no tiene, pide la dirección al usuario. No se envían comprobantes con una baja en curso.

## Dar de baja (anular)

La baja es **definitiva**. Úsala solo si el usuario lo pide expresamente.

1. Revisa el comprobante con `get-document`: debe estar aceptado y sin notas vigentes.
2. Pide el motivo (por ejemplo, "ERROR EN EL RUC DEL CLIENTE").
3. Llama a `void-document` con `"preview": true` y el motivo. Muestra al usuario qué se enviará (comunicación de baja para facturas, resumen diario para boletas) y que **no se puede deshacer**.
4. Solo con su visto bueno, repite con los mismos datos y sin `preview`; en **producción**, además `"confirm": true`.
5. Resultado:
   - `voided`: SUNAT aceptó la baja.
   - `in_process`: SUNAT aún la procesa; el sistema la consulta cada 5 minutos. Revisa más tarde con `get-document`.
   - `rejected`: el comprobante sigue vigente; explica el motivo.
   - `already_voided`: ya tenía una baja; no envíes otra.
- Si está **fuera de plazo** o tiene notas, la baja no procede: ofrece una **nota de crédito** con la skill `emitir-comprobante` (motivo 01, anulación de la operación).
- Por OAuth, solo el propietario o un administrador de la empresa puede dar de baja.
