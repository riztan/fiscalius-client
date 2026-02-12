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

## Impacto del Fix

- ✅ Corrige error de duplicación de monedas
- ✅ Permite edición correcta de registros nuevos
- ✅ Mantiene consistencia entre modelo local y servidor
- ✅ Mejora experiencia del usuario (elimina mensajes de error confusos)

---
*Este fix demuestra la importancia de mantener sincronizados los estados locales con los servidores en arquitecturas cliente-servidor.*