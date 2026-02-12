# Fix del Bug de Creación de Monedas en mae_monedas.xbs

## Problema Detectado

El programa `mae_monedas.xbs` presentaba un bug crítico al crear nuevas monedas:

1. **Síntoma**: Al crear una moneda nueva, el sistema emitía el mensaje "el registro ya existe" al intentar guardar cualquier cambio posterior
2. **Causa raíz**: El script no actualizaba el ID del modelo local después de guardar el registro nuevo
3. **Impacto**: El sistema siempre consideraba el registro como "nuevo", causando intentos de inserción duplicada

## Flujo del Problema

### Flujo Original (con bug):
1. Usuario填写 código y nombre → se crea registro nuevo en modelo local
2. Sistema llama a `Save()` → servidor inserta registro y asigna ID
3. **PROBLEMA**: El ID retornado por el servidor NO se actualizaba en el modelo local
4. Al intentar editar → sistema creía que seguía siendo nuevo
5. Nuevo `Save()` → servidor intentaba insertar duplicado → error "registro ya existe"

### Flujo Corregido:
1. Usuario填写 código y nombre → se crea registro nuevo
2. Sistema llama a `Save()` → servidor inserta y asigna ID
3. **SOLUCIÓN**: `hData := ::rMoneda:Data()` obtiene datos con ID
4. **SOLUCIÓN**: `oModel:Set(aIter, __ID, hb_ntos(hData["id"]))` actualiza modelo local
5. Al editar → sistema reconoce registro existente → actualización correcta

## Cambios Realizados (git diff)

### 1. Definición de columna ID
```diff
+   /* Columna ID     */
+   #define __ID       1
```
**Propósito**: Referenciar columna ID del TreeView para actualizaciones

### 2. Corrección Principal - Actualización de ID después de guardar
```diff
+            hData := ::rMoneda:Data()
+            if !Empty( hData["id"] )
+               oModel:Set( aIter, __ID, hb_ntos(hData["id"]) )
+            endif
```
**Propósito**: Sincronizar el modelo local con el ID real de la base de datos

### 3. Mejoras menores
- Inicialización de `::hMoneda := hb_hash()` para tracking
- Corrección de comentarios internos

## Análisis Técnico

### Clave del problema
Cuando `GetMoneda()` se ejecuta sin parámetros de ID, el servidor asume que es un registro nuevo. El script mantenía el modelo local sin ID, por lo que:

```xbase
// Al crear nuevo registro
::rMoneda := ::oParentForm:oGEmpresa:GetMoneda( {"codigo"=> ''} )
// Servidor recibe: { "codigo": "", "id": null } → INSERT

// Pero después de Save() el modelo local seguía sin ID
// → Próxima edición → mismo flujo de nuevo registro → DUPLICADO
```

### La solución
```xbase
// Después de guardar exitosamente
hData := ::rMoneda:Data()  // Obtiene datos con ID asignado por servidor
if !Empty( hData["id"] )
   oModel:Set( aIter, __ID, hb_ntos(hData["id"]) )  // Actualiza modelo local
endif

// Ahora GetMoneda() recibirá el ID → servidor hará UPDATE, no INSERT
```

## Lecciones Aprendidas

1. **Sincronización Modelo-Servidor**: Siempre mantener el modelo local sincronizado con el estado real en la base de datos
2. **IDs Generados por Servidor**: Cuando el servidor genera IDs, deben reflejarse inmediatamente en el modelo local
3. **Estado del Registro**: El sistema debe distinguir claramente entre "registro nuevo" y "registro existente"
4. **Testing de CRUD**: Probar completamente el ciclo Crear-Leer-Actualizar-Eliminar

## Advertencias para Desarrolladores

⚠️ **¡IMPORTANTE!** Este patrón de bug puede repetirse en otros scripts con:

- **TreeViews editables inline** que permiten crear nuevos registros
- **Modelos locales** que no se sincronizan con IDs del servidor
- **Callbacks de edición** que llaman a `Save()` sin actualizar estado

**Scripts revisar prioridad:**
- `mae_clientes2.xbs` - Similar estructura de TreeView editable
- `mae_inventario.xbs` - Probable patrón similar
- Cualquier script con `DEFINE MODEL` y `oRenderer:SetEditable(.T.)`

**Patrón a buscar:**
```xbase
// Después de Save() exitoso, buscar esta actualización:
hData := ::rObject:Data()
if !Empty( hData["id"] )
   oModel:Set( aIter, __ID, hb_ntos(hData["id"]) )
endif
```

## Mejora Sugerida - Helper Estandarizado

### Función Helper Principal
```xbase
FUNCTION __UpdateModelAfterSave( oModel, aIter, hObject, nIDCol )
   LOCAL hData := hObject:Data()
   if !Empty( hData["id"] )
      oModel:Set( aIter, nIDCol, hb_ntos(hData["id"]) )
   endif
RETURN .T.
```

### Implementación Completa con Validaciones
```xbase
FUNCTION __UpdateModelAfterSave( oModel, aIter, hObject, nIDCol, cLogContext )
   LOCAL hData, nID
   
   // Validaciones obligatorias
   if hb_isNIL(oModel) .or. hb_isNIL(aIter) .or. hb_isNIL(hObject)
      MsgAlert("Error: Parámetros inválidos en __UpdateModelAfterSave", "DEBUG")
      RETURN .F.
   endif
   
   // Obtener datos actualizados del objeto remoto
   hData := hObject:Data()
   if hb_isNIL(hData) .or. !hb_isHash(hData)
      if !Empty(cLogContext)
         MsgAlert("Error: No se pueden obtener datos del objeto en " + cLogContext, "DEBUG")
      endif
      RETURN .F.
   endif
   
   // Verificar si existe ID asignado
   if !hb_hHasKey(hData, "id") .or. Empty(hData["id"])
      if !Empty(cLogContext)
         // Es normal para registros nuevos que aún no tienen ID
         MsgInfo("Registro sin ID en " + cLogContext + " - posible creación nueva", "DEBUG")
      endif
      RETURN .T.
   endif
   
   // Extraer y validar ID
   nID := hData["id"]
   if hb_isNIL(nIDCol) .or. nIDCol <= 0
      MsgAlert("Error: Columna ID inválida en __UpdateModelAfterSave", "DEBUG")
      RETURN .F.
   endif
   
   // Actualizar modelo local
   oModel:Set( aIter, nIDCol, hb_ntos(nID) )
   
   // Log opcional para debugging
   if !Empty(cLogContext)
      MsgInfo("Modelo actualizado - ID: " + hb_ntos(nID) + " en " + cLogContext, "DEBUG")
   endif
   
RETURN .T.
```

### Implementación en Scripts Existentes
```xbase
// Al final del proceso de guardado (después de Save())
if !::rMoneda:Save() 
   MsgAlert("Se presenta problema para registrar los datos.", "Atención")
   return
endif

// Línea original reemplazada por helper
// hData := ::rMoneda:Data()
// if !Empty( hData["id"] )
//    oModel:Set( aIter, __ID, hb_ntos(hData["id"]) )
// endif

// Nueva implementación estandarizada
__UpdateModelAfterSave( oModel, aIter, ::rMoneda, __ID, "mae_monedas" )
```

### Helper para Validación Previa al Guardar
```xbase
FUNCTION __ValidateBeforeSave( hObject, aRequiredFields, cContext )
   LOCAL hData, cField
   
   if hb_isNIL(hObject)
      MsgAlert("Error: Objeto nulo en " + cContext, "VALIDACIÓN")
      RETURN .F.
   endif
   
   hData := hObject:Data()
   if hb_isNIL(hData)
      MsgAlert("Error: No hay datos en " + cContext, "VALIDACIÓN")
      RETURN .F.
   endif
   
   // Validar campos requeridos
   for each cField in aRequiredFields
      if !hb_hHasKey(hData, cField) .or. Empty(hData[cField])
         MsgAlert("Campo requerido vacío: " + cField + " en " + cContext, "VALIDACIÓN")
         RETURN .F.
      endif
   next
   
RETURN .T.
```

### Uso Combinado en Scripts
```xbase
// Validación previa al guardar
if !__ValidateBeforeSave(::rMoneda, {"codigo", "nombre"}, "mae_monedas")
   return
endif

// Proceso de guardado
if !::rMoneda:Save() 
   MsgAlert("Se presenta problema para registrar los datos.", "Atención")
   return
endif

// Actualización del modelo local
__UpdateModelAfterSave( oModel, aIter, ::rMoneda, __ID, "mae_monedas" )
```

### Beneficios del Helper
1. **Estandarización**: Mismo comportamiento en todos los scripts
2. **Validaciones**: Chequeos robustos de parámetros y datos
3. **Debugging**: Logs opcionales para troubleshooting
4. **Prevención**: Validaciones previas al guardar
5. **Mantenimiento**: Cambios en un solo lugar para todo el sistema

## Impacto del Fix

- ✅ Corrige error de duplicación de monedas
- ✅ Permite edición correcta de registros nuevos
- ✅ Mantiene consistencia entre modelo local y servidor
- ✅ Mejora experiencia del usuario (elimina mensajes de error confusos)

---
*Este fix demuestra la importancia de mantener sincronizados los estados locales con los servidores en arquitecturas cliente-servidor.*