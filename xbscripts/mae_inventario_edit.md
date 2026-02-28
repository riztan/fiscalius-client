# mae_inventario_edit.xbs - Editor de Inventario

## Descripción
Formulario de edición de productos del inventario en el cliente Fiscalius.

## Campos nuevos: capacidad y grados_alcohol

### UI (mae_inventario.ui)
Los campos están definidos en el archivo de recursos GTK:

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

### Transformación de datos
```xbscript
oInvForm:hData["capacidad"        ] := TRANSFORM( oInvForm:hData["capacidad"        ], P_92 )
```

### Columnas en Save()
```xbscript
FOR EACH cCol IN {"precio_unitario","stock_total","stock_minimo","imp_pvp","codigo_barras","capacidad","grado"}
```

### Base de datos
```sql
ALTER TABLE orseit.mae_inventario 
ADD COLUMN capacidad NUMERIC(10,2),
ADD COLUMN grados_alcohol NUMERIC(10,2);
```

## Propósito
Estos campos son útiles para:
- **capacidad**: Volumen de la botella/envase (ej: 750ml, 1L)
- **grado**: Porcentaje de alcohol (ej: 40% para whisky, 12% para vino)
