---
name: guias-de-remision
description: Emite guías de remisión electrónicas remitente (09) y transportista (31) ante SUNAT con Aion Facturación, y consulta las emitidas. Úsala cuando el usuario necesite trasladar mercadería, pida una guía de remisión, GRE, guía remitente o guía transportista, o pregunte por una guía ya emitida.
---

# Guías de remisión electrónicas

Herramientas del servidor `facturacion`: `company-info`, `search-customers`, `search-products`, `issue-despatch`, `list-despatches`, `get-despatch`, `get-document`, `retry-sunat`.

## Qué tipo de guía

- **Remitente (09)**, serie T001: la emite quien envía la mercadería (venta, compra, traslado entre establecimientos…). Es la más común.
- **Transportista (31)**, serie V001: la emite la empresa de transporte que lleva mercadería de otro.

Revisa con `company-info` que la empresa tenga la serie y las credenciales de guías. Si faltan, explica que se configuran en el panel (**Configuración → SUNAT y certificado**, credenciales de guías).

## Datos que hay que reunir (pregunta los que falten; no los inventes)

1. **Fecha de inicio del traslado** (`transfer_date`, desde hoy).
2. **Motivo** (solo 09, catálogo 20): `01` venta, `02` compra, `04` traslado entre establecimientos de la misma empresa, `13` otros.
3. **Comprobante relacionado** (09 por venta): la factura o boleta aceptada que sustenta el traslado, en `related_document` (por ejemplo F001-15). Si la venta aún no tiene comprobante, emítelo antes con la skill `emitir-comprobante`.
4. **Destinatario** (`recipient`): en el motivo 04 es la propia empresa.
5. **Punto de partida y de llegada**: dirección y **ubigeo INEI de 6 dígitos** de cada uno.
6. **Bienes**: productos del catálogo (`search-products`) o descripción, unidad y cantidad; y el **peso bruto total**.
7. **Modalidad de transporte** (solo 09):
   - `01` **público**: datos del transportista en `carrier` (RUC, razón social y registro MTC).
   - `02` **privado**: vehículos (placa) en `vehicles` y conductores (documento, nombres, apellidos y licencia) en `drivers`.
8. **Solo 31**: remitente (`sender`), documentos relacionados, quién paga el flete (`freight_payer`) y los indicadores si aplican.

## Emitir

1. Usa una `external_reference` única para la guía y reutilízala si repites.
2. Llama a `issue-despatch` con `"preview": true` y muestra al usuario un resumen: tipo, motivo, partida y llegada, bienes, peso, transporte y fecha.
3. Con su visto bueno, emite con los mismos datos: en beta sin `preview` (en beta la guía solo se firma, no se envía a SUNAT); en **producción** con `"confirm": true`.
4. Da el número de la guía, su estado y los enlaces `links` copiados tal cual, más el enlace `panel`.
5. Si SUNAT no respondió, el sistema reintenta; también sirve `retry-sunat` con `despatch_id`.

## Consultar

`list-despatches` (por fechas, estado o texto) y `get-despatch` (por id, número o referencia). Los enlaces vencen a los 30 minutos.
