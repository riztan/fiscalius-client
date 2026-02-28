# Documentación de vta_imprimir.xbs


## Cambios Realizados

### 4. Agregado campos capacidad y grados_alcohol en impresión de factura

Se agregaron los campos de capacidad y grados de alcohol del producto en el detalle de la factura impresa.

**Origen de datos**: 
- Los campos vienen de `tvfactura.prg` en el método GetData()
- Se obtienen del join con `mae_inventario`

```xbscript
" mi.capacidad, "+;
" mi.grados_alcohol "+;
```

**Propósito**:
- Mostrar volumen de la botella (ej: 750ml, 1L)
- Mostrar porcentaje de alcohol (ej: 40% para whisky, 12% para vino)
- Cumplir con requisitos de facturación para bebidas alcohólicas

### 1. Agregado campo "Impuesto al PVP" en el detalle de impresión de factura

Se agregó el campo de impuesto al precio de venta público (PVP) en el detalle de la factura impresa.

**Ubicación**: Detalle de productos en el Excel

---

### 2. Conversión de totales de factura para Excel

Se convirtieron los valores de los totales de la factura para que el archivo Excel los lea correctamente.

**Problema anterior**: Los valores del MONEY ENTRY (`::oEntTotal`) retornan strings con formato monetario, lo cual causaba errores al intentar usarlos en operaciones matemáticas.

**Solución aplicada**:
```xbase
// Antes (incorrecto):
xlsx_set_string ::worksheet cell: "N27" value: ::oEntTotal:GetValue()

// Después (correcto):
xlsx_set_string ::worksheet cell: "N27" value: hb_ntos( VAL( oForm:oEntTotal:GetValue() ) )
```

**Explicación**:
- `oForm:oEntTotal:GetValue()` - Obtiene el valor del campo MONEY ENTRY
- `VAL()` - Convierte el string a número
- `hb_ntos()` - Convierte el número de vuelta a string para Excel

---

### 3. Validación de código de producto: Unidades vs Cajas

Se implementó validación según los últimos 2 caracteres del código de producto para determinar si es **unidad** o **caja**:

- Si los últimos 2 caracteres son `"01"` → Es **UNIDAD**
- Si los últimos 2 caracteres son distintos de `"01"` → Es **CAJA**

**Lógica implementada**:
```xbase
// Verificar si es unidad (terminación 01)
if RIGHT(aLine[3], 2) == "01"
   // Es unidad
else
   // Es caja
endif
```

**Campos afectados**:
- Columna "Caj." - Cantidad de cajas
- Columna "Uni." - Cantidad de unidades  
- Columna "Tot. Uni." - Total de unidades (cajas * unidades por caja + unidades sueltas)

**Ejemplo de código**:
```xbase
nUniCaj := IIF(RIGHT(hb_ntos(aLine[3]),2)="01", 0, VAL(RIGHT(aLine[3],2)))
nUni := iif(RIGHT(aLine[3], 2) == "01", 0, aLine[5])
nCaj := IIF(RIGHT(aLine[3], 2) == "01", 0, aLine[5])
nTotUniLi := nCaj * nUniCaj + nUni
```

---

## Estructura del Excel generado

| Columna | Descripción |
|---------|-------------|
| B | Código del producto |
| D | Descripción |
| I | Cantidad en Cajas |
| J | Cantidad en Unidades |
| K | Total Unidades |
| L | Precio Unitario |
| N | Precio Total |

---

## Notas Técnicas

- El MONEY ENTRY en TPuy retorna strings con formato monetario
- Es necesario usar `VAL()` para convertir a número antes de operaciones
- Los últimos 2 caracteres del código determinan el tipo de presentación (01 = unidad)

---

*Documentación actualizada automáticamente*
