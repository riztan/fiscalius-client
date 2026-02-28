# vta_facturas.xbs - Correcciones Realizadas

## Fecha
28/02/2026

## Problema Original
El cálculo de IVA no discriminaba correctamente entre los diferentes porcentajes:
- IVA reducido (8%)
- IVA general (16%)
- Productos exentos (0%)

## Cambios Realizados

### 1. Verificación de porcentaje > 0
Se agregó validación `if nPorcentaje>0` antes de calcular el monto base para productos con IVA.

```xbscript
nPorcentaje := ToNum( oTView:GetAutoValue( __IVAPORCEN, aIter ) )
if nPorcentaje>0
   nMonto := ToNum( oTView:GetAutoValue( __SUBTOTAL, aIter ) )
   nBase  := ToNum( ::oEntMonBase:GetValue() )
   nOtrImp:= ToNum( ::oEntOtrImp:GetValue())
   nTotal := ROUND( nBase+nMonto, 2 )
   ::oEntMonBase:SetValue( TRANSFORM( nTotal, P_92 ) )
endif
```

### 2. Productos Exentos de IVA (0%)
Manejo separado para productos con `nPorcentaje = 0`:
- Se asigna a `oEntIvaE` (campo exento)

```xbscript
if nPorcentaje = 0  // Producto exento de IVA
   nMonto := ToNum( oTView:GetAutoValue( __SUBTOTAL, aIter ) )
   nBase := ToNum( ::oEntIvaE:GetValue() )
   nTotal := ROUND( nBase + nMonto, 2 )
   ::oEntIvaE:SetValue( TRANSFORM( nTotal, P_92 ) )
endif
```

### 3. IVA Reducido (8%) vs IVA General (16%)
Separación correcta según el porcentaje:
- `nPorcentaje = 8` → `oEntIvaR` (IVA reducido)
- `nPorcentaje = 16` → `oEntIvaG` (IVA general)

```xbscript
if nPorcentaje > 0
   nMonto := ToNum( oTView:GetAutoValue( __IVAMONTO, aIter ) )
   if nPorcentaje = 8  // IVA Reducido
      nBase  := ToNum( ::oEntIvaR:GetValue() )
      nTotal := ROUND( nBase+nMonto, 2 )
      ::oEntIvaR:SetValue( TRANSFORM( nTotal, P_92 ) )
   elseif nPorcentaje = 16  // IVA General
      nBase  := ToNum( ::oEntIvaG:GetValue() )
      nTotal := ROUND( nBase+nMonto, 2 )
      ::oEntIvaG:SetValue( TRANSFORM( nTotal, P_92 ) )
   endif
endif
```

## Campos Involucrados
- `oEntMonBase` - Monto base/subtotal
- `oEntIvaE` - IVA Exento (0%)
- `oEntIvaR` - IVA Reducido (8%)
- `oEntIvaG` - IVA General (16%)
- `oEntOtrImp` - Otros impuestos

---

## Campo Total en Bs (Bolívares)

### Definición del campo
```xbscript
::nMonTotBs := 0
DEFINE MONEY ENTRY ::oEntTotBs VAR ::nMonTotBs ;
      ID "ent_montotbs" RESOURCE ::oRes 
::oEntTotBs:Justify( GTK_JUSTIFY_RIGHT )
::oEntTotBs:SetEditable( .F. )
```

### Inicialización
```xbscript
::oEntTotBs:SetValue( "0" )
```

### Cálculo del total en Bs
El total en bolívares se calcula multiplicando el monto total por la tasa de cambio:

```xbscript
// Carga de datos existentes
::oEntTotBs:SetValue( TRANSFORM(::hFactura["monto_total"]*TONUM(SUBSTR(::cDivisa,5)),P_92) )

// Al calcular totales
::oEntTotBs:SetValue( TRANSFORM( nTotal*TONUM(SUBSTR(::cDivisa,5)), P_92 ) )
```

### Propósito
- Mostrar el monto total de la factura en bolívares (Bs)
- Utiliza la tasa de cambio de la divisa (campo `::cDivisa`)
- Campo de solo lectura (`SetEditable(.F.)`)
