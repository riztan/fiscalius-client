# vta_notas_cd.xbs - Correcciones de IVA

## Fecha
28/02/2026

## Problema Original
El cálculo de IVA no discriminaba correctamente entre los diferentes porcentajes. Todo IVA se asignaba al campo general (oEntIvaG) sin importar si era 8% o 16%.

## Cambios Realizados

### 1. Agregada variable nPorcentaje
```xbscript
local nBase, nMonto, nTotal, cIva, nPorcentaje
nPorcentaje := ToNum( oTView:GetAutoValue( __IVAPORCEN, aIter ) )
```

### 2. Cálculo de Base Imponible solo si hay IVA (> 0)
Se evitó duplicar el cálculo del subtotal/base imponible. Solo se calcula si el producto tiene IVA (> 0).

```xbscript
// Calcular monto base si hay IVA
if nPorcentaje > 0
   nMonto := ToNum( oTView:GetAutoValue( __SUBTOTAL, aIter ) )
   nBase  := ToNum( ::oEntMonBase:GetValue() )
   nTotal := ROUND( nBase + nMonto, 2 )
   ::oEntMonBase:SetValue( TRANSFORM( nTotal, P_92 ) )
endif
```

### 3. Productos Exentos de IVA (0%)
Manejo separado para productos con `nPorcentaje = 0`:

```xbscript
// Manejar productos exentos de IVA (0%)
if nPorcentaje = 0
   nMonto := ToNum( oTView:GetAutoValue( __SUBTOTAL, aIter ) )
   nBase  := ToNum( ::oEntIvaE:GetValue() )
   nTotal := ROUND( nBase + nMonto, 2 )
   ::oEntIvaE:SetValue( TRANSFORM( nTotal, P_92 ) )
endif
```

### 4. IVA Reducido (8%) vs IVA General (16%)
Separación correcta según el porcentaje:

```xbscript
// Calcular monto IVA y separar por porcentaje
if nPorcentaje > 0
   nMonto := ToNum( oTView:GetAutoValue( __IVAMONTO, aIter ) )
   if nPorcentaje = 8  // IVA Reducido
      nBase  := ToNum( ::oEntIvaR:GetValue() )
      nTotal := ROUND( nBase + nMonto, 2 )
      ::oEntIvaR:SetValue( TRANSFORM( nTotal, P_92 ) )
   elseif nPorcentaje = 16  // IVA General
      nBase  := ToNum( ::oEntIvaG:GetValue() )
      nTotal := ROUND( nBase + nMonto, 2 )
      ::oEntIvaG:SetValue( TRANSFORM( nTotal, P_92 ) )
   endif
endif
```

## Campos Involucrados
- `oEntMonBase` - Monto base/subtotal (solo productos con IVA > 0)
- `oEntIvaE` - IVA Exento (0%)
- `oEntIvaR` - IVA Reducido (8%)
- `oEntIvaG` - IVA General (16%)
- `oEntOtrImp` - Otros impuestos

## Notas
- Se eliminó el cálculo duplicado de subtotal/base imponible que causaba doble suma
- El hash `hDocumento["iva"]` sigue guardando los datos de IVA por porcentaje
