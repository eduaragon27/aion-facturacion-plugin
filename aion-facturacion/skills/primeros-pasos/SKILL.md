---
name: primeros-pasos
description: Conecta Claude con Aion Facturación y revisa si la empresa ya puede emitir comprobantes electrónicos ante SUNAT. Úsala cuando el usuario pregunte cómo empezar, cómo conectar su facturación, si su empresa está lista para facturar, en qué entorno está (beta o producción) o qué series tiene.
---

# Primeros pasos con Aion Facturación

Aion Facturación emite comprobantes electrónicos ante SUNAT (Perú) para **una empresa**. Este plugin conecta Claude con el servidor MCP `facturacion`. Responde siempre en español.

## 1. Comprobar la conexión

Llama a la herramienta `company-info`.

- **Si no hay conexión o pide autorización**: explica al usuario que debe autorizar el acceso. En Claude Code, que escriba `/mcp`, elija `facturacion` y pulse *Authenticate*. Se abre Aion Facturación: inicia sesión, elige la empresa y el nivel de acceso:
  - *Solo consultar*: ver empresa, clientes, productos, comprobantes y reportes.
  - *Consultar y registrar*: además crear o editar clientes y productos, y registrar cotizaciones.
  - *También emitir*: además emitir comprobantes, guías, bajas y correos (también enviar y convertir cotizaciones).
- Si el usuario aún no tiene cuenta, que se registre en https://facturacion.aionstoreperu.com (empieza en beta, gratis y sin valor tributario).
- Si su programa no admite OAuth, puede crear un token en **Integraciones → Agente de IA** y usarlo como cabecera `Authorization: Bearer <token>`.

## 2. Resumir el estado de la empresa

Con la respuesta de `company-info`, cuéntale al usuario en pocas líneas:

1. **Entorno**: en **beta** los comprobantes son de prueba (no valen ante SUNAT); en **producción** son reales y las emisiones piden confirmación.
2. **Régimen**: el Nuevo RUS solo emite boletas.
3. **Documentos que emite y sus series** (por ejemplo F001 facturas, B001 boletas, FC01 notas de crédito).
4. **Lo que falta para emitir** (dirección fiscal, certificado, series, usuario SOL). Si falta algo, dile dónde completarlo en el panel: **Configuración → Empresa y sucursales**, **Comprobantes y series** o **SUNAT y certificado**.
5. **Licencia**: cupo de comprobantes del mes y, en beta, el tope de pruebas.
6. **Módulos**: si aparecen las herramientas de notas de venta, la empresa tiene ese módulo.

## 3. Proponer el siguiente paso

Según lo que pida el usuario, sigue con la skill que corresponda:

- Emitir una factura, boleta o nota → `emitir-comprobante`.
- Ver ventas, buscar un comprobante, reenviarlo por correo o anularlo → `consultar-ventas`.
- Guía de remisión → `guias-de-remision`.
- Nota de venta (documento interno) y convertirla en boleta o factura → `notas-de-venta`.
- Cotización, presupuesto o proforma para un cliente, enviarla y convertirla cuando la acepte → `cotizaciones`.

## Reglas que valen siempre

- No inventes RUC, DNI, precios ni números de comprobante. Pregunta o búscalos.
- El servicio calcula IGV, totales, monto en letras y correlativo: no los calcules tú.
- Los enlaces de `links` (PDF, XML, CDR) vencen a los 30 minutos: cópialos exactamente como llegan y compártelos solo con el usuario. Da también el enlace `panel`.
- Si una herramienta devuelve un error, léelo: viene en español con el campo exacto que hay que corregir.
- **Error de licencia** (campo license: cupo agotado, licencia vencida o términos sin aceptar): no es un dato mal puesto, así que no lo reintentes ni cambies la referencia. Explícale al usuario el mensaje y dónde resolverlo: **Configuración → Licencia y módulos**. Una subida a un plan de mayor cupo se activa apenas la aprueban; con el cupo agotado aún puede emitir notas de crédito y débito para corregir lo ya emitido, y consultar, descargar y reenviar todo.
