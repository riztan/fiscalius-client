# AGENTS.md - Fiscalius Client

**Proyecto**: Cliente de escritorio TPuy para el ERP fiscal venezolano Fiscalius  
**Directorio**: `/home/riztan/git/fiscalius-client`

---

## Build / Compile

```bash
# Compile with xHarbour (requires TPuy framework)
# Typically run via tpuy or hbbuild
```

---

## Code Style - Harbour/xHarbour

### Regla Crítica - LOCAL en Harbour
- **LOCAL debe declararse AL INICIO del procedimiento**, antes de cualquier código ejecutable
- **NUNCA** dentro de FOR EACH, IF, WHILE, TRY, etc.
- Si necesitas variables dentro de un bucle, decláralas al inicio del procedimiento

```harbour
// ✅ Correcto
procedure MiProc()
   local aLine, nLen, aVal  // LOCAL al inicio

   FOR EACH aLine IN aData
      nLen := Len( aLine )   // Uso sin declaración
      aVal := {}
      ...
   NEXT
end

// ❌ Incorrecto - Error E0004
procedure MiProc()
   FOR EACH aLine IN aData
      local nLen := Len( aLine )  // LOCAL dentro del bucle
      ...
   NEXT
end
```

### Estructura de Archivos .xbs
```harbour
/** ModuleName
 *  archivo.xbs   Description
 */

#include "tpy_xbs.ch"

// Procedimientos
procedure vta_facturas( oFormParent, cTipoDocumento )
   local oForm, oModel
   local aData
   ...
return

procedure __vtaFactura( oForm, oFormParent, oBoxFactura, oModel, lNew )
   ...
return
```

### Naming Conventions
| Elemento | Formato | Ejemplo |
|----------|---------|---------|
| Procedimientos | snake_case | `vta_facturas()`, `__Calcula()` |
| Variables LOCAL | camelCase | `cQry`, `nId`, `aData`, `hData` |
| Constantes | UPPER_CASE | `CB_ESTADO_ABIERTO` |
| Archivos .xbs | snake_case | `vta_facturas.xbs`, `vta_notas_entrega.xbs` |

### Tipos de Datos
```harbour
LOCAL cQry, cValor    // String
LOCAL oQry           // Objeto Query
LOCAL nId             // Numeric
LOCAL hData           // Hash (hb_hash)
LOCAL aItems          // Array
LOCAL lNew            // Logical (.T. / .F.)
LOCAL dFecha          // Date
```

---

## Error Handling

### TRY/CATCH
```harbour
TRY
   // código que puede fallar
   ::oModDetal:oTreeView:ClearModel(.t.)
CATCH oErr
   View( "Error: " + oErr:Description )
END
```

### Validación de Entrada
```harbour
IF empty( hData["cedula"] )
   RETURN .F.
ENDIF

IF hb_isNumeric( nMonto ) .and. nMonto <= 0
   RETURN .F.
ENDIF
```

---

## LIST_STORE - Errores Comunes

### IMPORTANTE: LIST_STORE espera strings
El widget `APPEND LIST_STORE` espera **todos los valores como strings de texto**. No realizar operaciones matemáticas dentro de VALUES.

```harbour
// ❌ INCORRECTO - intentar dividir strings
APPEND LIST_STORE ::oModDetal:oLbx ITER aIter ;
       VALUES aLine[21], aLine[21]/aLine[5], aLine[5], aLine[22], aLine[22]/aLine[5]

// ✅ CORRECTO - pasar valores directamente (ya formateados como strings)
// Los valores numéricos deben estar convertidos a string ANTES de llegar aquí
APPEND LIST_STORE ::oModDetal:oLbx ITER aIter ;
       VALUES aLine[21], aLine[22], aLine[23], aLine[24], aLine[25]
```

### Conversión de tipos
- Usar `hb_ntos()` para convertir números a strings
- No hacer operaciones matemáticas dentro de VALUES
- Los datos ya deben venir formateados del servidor

### Verificar longitud de arrays
Antes de acceder a índices de array, verificar que existan:

```harbour
nLen := LEN( aLine )
cValor := iif(nLen >= 21, aLine[21], "0")  // Verificar antes de acceder
```

---

## Architecture

### Estructura de Archivos
```
fiscalius-client/
├── xbscripts/        // Scripts principales (vta_*.xbs)
├── ui/               // Archivos .ui de GTK
├── docs/             // Documentación
└── install/          // Scripts de instalación
```

### Módulos Principales
| Archivo | Descripción |
|---------|-------------|
| `vta_facturas.xbs` | Módulo de facturas |
| `vta_notas_cd.xbs` | Notas de crédito/débito |
| `vta_notas_entrega.xbs` | Notas de entrega |
| `vta_pedidos.xbs` | Pedidos |

---

## Git Workflow
1. No hacer commits sin autorización
2. **NUNCA hacer push sin autorización explícita del usuario**
3. **Probar siempre los cambios localmente antes de hacer commit**
4. Working directory limpio antes de operar
5. **Antes de push, informar "listo para probar" y esperar confirmación**

---

## Aprendidos - Módulo vta_notas_entrega

### Problemas comunes y soluciones

1. **Error E0004 - LOCAL declaration follows executable statement**
   - LOCAL debe declararse siempre al inicio del procedimiento
   - Antes de editar, verificar las primeras líneas del procedimiento para ver qué LOCAL ya existen

2. **Estructura de modelo de detalles**
   - Notas de entrega usa 23 columnas (como notas_cd): detalle_id, documento, código, descripción, cantidad, precio, descuento, subtotal, iva_porcentaje, iva_monto, total, nombre_global, tipo_inventario, lote, fecha_vcmto, ubicacion, almacen_id, nombre_almacen, detalles_json, item, imp_pvp, imp_pvp_unit, cant_maxima

3. **Carga de datos desde Detal()**
   - Usar exactamente el mismo patrón que vta_notas_cd.xbs
   - No usar bucles individuales con AADD - usar Append directo
   - Para cant_maxima, usar `hb_ntos(aLine[5])` (repetir la cantidad)

4. **Validación de cantidad máxima**
   - La columna 23 es cant_maxima
   - Usar `oTreeView:GetAutoValue( 23 )` para obtener el valor

5. **Selección de facturas**
   - Usar `::rDocumento:Facturas()` del servidor (via VNOTAS_CD:Facturas)
   - No usar `rFacturas:Filter()` en cliente - no funciona así

6. **Error "condicional: nil"**
   - Ocurre cuando se accede a propiedades de objetos nulos
   - Verificar siempre que el objeto existe antes de acceder a sus propiedades

### Diferencias entre vta_notas_cd y vta_notas_entrega
- notas_cd tiene retención IVA e ISLR, nota_entrega NO
- El resto del código es prácticamente identical
- El modelo de datos es el mismo (23 columnas)

---

## Pendiente: Facturas con saldo para notas de entrega

### Problema
El método actual `VNOTAS_CD:Facturas()` retorna todas las facturas del cliente (usando vista `vw_facturas_notas_saldo`), pero necesitamos filtrar solo las que tienen saldo por relacionar - es decir, facturas donde NO todos sus detalles han sido completamente relacionados con notas de entrega anteriores.

### Lógica requerida
1. Filtrar por cliente_id de la nota de entrega
2. Solo facturas con estado = 'C' (cerradas)
3. Para cada detalle de la factura:
   - cantidad_original = detalle.cantidad
   - cantidad_relacionada = SUM(cantidades de notas de entrega relacionadas a este detalle)
   - MOSTRAR la factura si: cantidad_relacionada < cantidad_original
4. Ocultar la factura solo cuando TODOS sus detalles han sido relacionados en su TOTALIDAD

### Propuesta de solución
Crear un método nuevo en `VNOTAS_CD` (servidor) - algo como `FacturasConSaldo()` - que ejecute una consulta SQL con la lógica above.

### Consulta SQL propuesta (sin probar aún)
```sql
SELECT DISTINCT t.id, t.numero, t.fecha, t.cliente_id, t.monto_total
FROM orseit.vta_transacciones t
JOIN orseit.vta_documentos d ON t.tipo_id = d.id
JOIN orseit.vta_transacciones_detalles td ON t.id = td.documento_id
WHERE d.id = 4  -- tipo factura
  AND t.estado = 'C'
  AND t.cliente_id = :cliente_id
  AND NOT EXISTS (
    SELECT 1 FROM orseit.vta_transacciones_detalles td2
    WHERE td2.documento_id = t.id
    AND (
      SELECT COALESCE(SUM(tdn.cantidad), 0)
      FROM orseit.vta_transacciones tne
      JOIN orseit.vta_documentos dne ON tne.tipo_id = dne.id
      JOIN orseit.vta_transacciones_detalles tdn ON tne.id = tdn.documento_id
      WHERE tne.doc_referencia_id = t.numero
        AND tne.estado = 'C'
        AND dne.id = 14  -- tipo nota de entrega (E)
        AND tdn.mae_inventario_id = td2.mae_inventario_id
    ) >= td2.cantidad
  )
ORDER BY t.numero
```

### Estado actual
- Pendiente probar la consulta en el servidor
- Pendiente crear el método en VNOTAS_CD (servidor)
- Pendiente actualizar cliente para usar el nuevo método

> **NOTA**: Esta sección "Pendiente: Facturas con saldo" es TEMPORAL - debe eliminarse una vez resuelto el tema.