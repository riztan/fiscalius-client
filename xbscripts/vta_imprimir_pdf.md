# vta_imprimir_pdf.xbs - Mejoras en Formato de Factura

## Fecha
02/04/2026

## Descripción
Se realizaron mejoras significativas en el formato de impresión de facturas PDF.

---

## Cambios Realizados

### 1. Detalle de Impuestos Separados
Se agregó el desglose de impuestos por tipo de IVA:

| Tipo | Descripción |
|------|-------------|
| Exento | Base exenta (0% IVA) |
| Base Imponible (G) | Base general (16% IVA) |
| Base Imponible (R) | Base reducido (8% IVA) |
| Alicuota IVA (16%) | Monto IVA general |
| AlicuotaIVA (8%) | Monto IVA reducido |

### 2. Campo IGTF (Impuesto a las Transacciones en Moneda Extranjera)
Se agregó la condición de pago para mostrar aviso de IGTF:

```xbscript
IF ::hFactura["condicion_id"] > 1
   oPDF:CMSAY( nY, 1.4, "Esta factura estará sujeta al cobro del 3% por concepto de IGTF en caso de ser pagada en una moneda distinta a la de curso legal en el país..." )
ENDIF
```

### 3. Capacidad y Grados de Alcohol
Se agregaron columnas en el detalle de la factura:

```xbscript
oPDF:CMSAY( nY, 12.2, "Cap." )    // Capacidad del producto
oPDF:CMSAY( nY, 13.2, "Grados" ) // Grados de alcohol
```

### 4. Detalle de Cobros
Se agregó sección de detalle de cobros/pagos:

```xbscript
local rDetCobro := ::rVentas:DetCobroFact( {...} )
local aDetCobro := rDetCobro:GetData()
```

---

## Campos Impresos

### Encabezado
- Número de factura
- Fecha
- Fecha de vencimiento

### Cliente
- Código/Nombre
- RIF
- Teléfono
- Dirección

### Detalle del Producto
- Código
- Descripción
- Cantidad
- Capacidad (nuevo)
- Grados Alcohol (nuevo)
- Precio Unitario
- Total
- % IVA
- Impuesto Pvp

### Totales
- SubTotal
- Exento
- Base Imponible (G)
- Base Imponible (R)
- IVA General (16%)
- IVA Reducido (8%)
- Otros Impuestos
- Total

### Condiciones
- Tipo de Cambio BCV
- IGTF (si aplica)

---

## Archivos Modificados
- `xbscripts/vta_imprimir_pdf.xbs`

## Notas
- El formato es Landscape (horizontal)
- Usa la librería HaruPDF
- Soporta múltiples líneas de productos
- Muestra detalle de cobros al pie
