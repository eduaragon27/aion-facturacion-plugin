# Plugins de Aion Media para Claude

Emite comprobantes electrónicos ante **SUNAT (Perú)** conversando con Claude: facturas, boletas, notas de crédito y débito, guías de remisión y notas de venta. Claude consulta tu empresa, busca clientes y productos, te muestra una **vista previa con los totales** y solo emite con tu visto bueno.

> «Emite una boleta para Ana Quispe, DNI 01234567, por 2 cajas de paracetamol.»

## Qué incluye

| Skill | Para qué |
| --- | --- |
| `primeros-pasos` | Conectar tu cuenta y revisar si tu empresa ya puede emitir (entorno, series, lo que falta). |
| `emitir-comprobante` | Facturas, boletas, notas de crédito y de débito, con validaciones y vista previa. |
| `consultar-ventas` | Reporte de ventas, buscar comprobantes, enviarlos por correo, reintentar y dar de baja. |
| `guias-de-remision` | Guías de remisión remitente (09) y transportista (31). |
| `notas-de-venta` | Notas de venta internas y su conversión en boleta o factura. |

El plugin conecta el servidor MCP de Aion Facturación (`https://facturacion.aionstoreperu.com/mcp`). El servicio calcula el IGV, los totales y el correlativo, y aplica las mismas reglas que la web: lo que no se puede hacer en la web tampoco se puede hacer desde Claude.

## Requisitos

- Una cuenta en [Aion Facturación](https://facturacion.aionstoreperu.com). Empiezas en **beta** (comprobantes de prueba, sin valor tributario); para emitir de verdad, pasas tu empresa a producción con tu certificado digital y tu usuario SOL secundario.
- Claude Code, o Claude (web y escritorio) con conectores.

## Instalar en Claude Code

```bash
/plugin marketplace add eduaragon27/aion-facturacion-plugin
/plugin install aion-facturacion@aion
```

Después escribe `/mcp`, elige `facturacion` y pulsa **Authenticate**. Se abre Aion Facturación: inicia sesión, elige la empresa y el nivel de acceso (solo consultar; consultar y registrar; o también emitir). Puedes quitar el acceso cuando quieras en **Integraciones → Agente de IA → Aplicaciones conectadas**.

## Usar en Claude (web o escritorio)

En **Configuración → Conectores**, agrega un conector personalizado con la dirección `https://facturacion.aionstoreperu.com/mcp` (deja vacíos el ID y el secreto de cliente) y autoriza tu empresa.

## Seguridad

- En **producción**, cada emisión, conversión o baja devuelve primero una vista previa; Claude solo la envía a SUNAT si tú lo confirmas.
- Cada venta lleva una referencia única: si algo se repite, recibes el mismo comprobante y no se duplica.
- Los enlaces a PDF, XML y CDR vencen a los 30 minutos.
- Tu clave SOL y tu certificado nunca pasan por Claude: quedan cifrados en Aion Facturación.

## Privacidad

El plugin no guarda datos: envía tus pedidos al servicio de Aion Facturación, que trata los datos de tu empresa y de tus clientes según su [política de privacidad](https://panel.aionstoreperu.com/privacidad) y sus [términos](https://panel.aionstoreperu.com/terminos).

## Soporte

Escríbenos desde [aionstoreperu.com](https://aionstoreperu.com).

## Licencia

MIT
