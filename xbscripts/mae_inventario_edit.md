# mae_inventario_edit.xbs - Editor de Inventario

## Descripción
Formulario de edición de productos del inventario en el cliente Fiscalius.

**Estado**: ✅ Implementado (18/03/2026)

---

## Campos: capacidad y grados_alcohol

### UI (mae_inventario.ui)
```xml
<object class="GtkEntry" id="ent_capacidad"></object>
<object class="GtkEntry" id="ent_grados_alcohol"></object>
```

### Entry Fields (mae_inventario_edit.xbs)

#### Campo capacidad
```xbscript
DEFINE INTEGER ENTRY oInvForm:oEntCapacidad VAR oInvForm:hData["capacidad"] ;
     ID "ent_capacidad"  RESOURCE oRes
```

#### Campo grados_alcohol
```xbscript
DEFINE INTEGER ENTRY oInvForm:oEntGradosAlcohol VAR oInvForm:hData["grados_alcohol"] ;
     VALID __ValidaMonto( oInvForm:oEntGradosAlcohol ) ;
     ID "ent_grados_alcohol"  RESOURCE oRes
```

### Propósito
- **capacidad**: Volumen de la botella/envase (ej: 750ml, 1L)
- **grados_alcohol**: Porcentaje de alcohol (ej: 40% para whisky, 12% para vino)

---

## Campos: unidad_ind, unidad_agru, cant_und_agru

### UI (mae_inventario.ui)
```xml
<object class="GtkEntry" id="ent_unidad_ind"></object>
<object class="GtkEntry" id="ent_unidad_agru"></object>
<object class="GtkEntry" id="ent_cant_und_agru"></object>
```

### Entry Fields (mae_inventario_edit.xbs)

#### Campo unidad_ind (Unidad Individual)
```xbscript
DEFINE INTEGER ENTRY oInvForm:oEntUnidadInd VAR oInvForm:hData["unidad_ind"] ;
     ID "ent_unidad_ind"  RESOURCE oRes
```

#### Campo unidad_agru (Unidad Agrupada)
```xbscript
DEFINE INTEGER ENTRY oInvForm:oEntUnidadAgru VAR oInvForm:hData["unidad_agru"] ;
     ID "ent_unidad_agru"  RESOURCE oRes
```

#### Campo cant_und_agru (Cantidad por Unidad Agrupada)
```xbscript
DEFINE INTEGER ENTRY oInvForm:oEntCantUndAgru VAR oInvForm:hData["cant_und_agru"] ;
     ID "ent_cant_und_agru"  RESOURCE oRes
```

### Transformación de datos
```xbscript
oInvForm:hData["cant_und_agru"] := TRANSFORM( oInvForm:hData["cant_und_agru"], P_92 )
```

### Propósito
- **unidad_ind**: Unidad individual de venta (ej: "UNIDAD", "PIEZA", "BOLSA")
- **unidad_agru**: Unidad de agrupación/presentación (ej: "CAJA", "PACK", "BANDEJA")
- **cant_und_agru**: Cantidad de unidades individuales por unidad agrupada (ej: 12 para una caja de 12 unidades)

### Ejemplo
| unidad_ind | unidad_agru | cant_und_agru | Significado |
|------------|-------------|---------------|-------------|
| UNIDAD | CAJA | 12 | Venta por unidad, presentación en cajas de 12 |
| UNIDAD | PACK | 6 | Venta por unidad, pack de 6 unidades |

---

## Columnas en Save()
```xbscript
FOR EACH cCol IN {"precio_unitario","stock_total","stock_minimo","imp_pvp","codigo_barras","capacidad","grados_alcohol","cant_und_agru"}
```

---

## Base de datos
```sql
ALTER TABLE orseit.mae_inventario 
ADD COLUMN IF NOT EXISTS capacidad NUMERIC(10,2),
ADD COLUMN IF NOT EXISTS grados_alcohol NUMERIC(10,2),
ADD COLUMN IF NOT EXISTS unidad_ind VARCHAR(20),
ADD COLUMN IF NOT EXISTS unidad_agru VARCHAR(20),
ADD COLUMN IF NOT EXISTS cant_und_agru NUMERIC(6,3);
```
