# Resumen de Sesión de Trabajo - Mayo 2026

## Problemas Identificados

### 1. Error BASE/1132 - Acceso a hash en cliente
- **Causa**: El servidor retornaba `perfil` como hash vacío `{}` cuando no había datos
- **Síntoma**: Error al acceder a `["perfil"]["desarrollo"]` porque el hash estaba vacío
- **Solución**: En `tusuario.prg` agregar valores por defecto cuando perfil está vacío o es NIL

### 2. Error hb_hGetDef no existe en Harbour
- **Causa**: Se usó función de xHarbour que no existe en Harbour de TPuy
- **Archivos**: `netio_login.xbs`, `empresa.prg`, `ventas.prg`
- **Solución**: Usar `hb_HHasKey()` + acceso directo al hash

### 3. Error ENDMETHOD no existe en xHarbour
- **Causa**: Se usó palabra clave que no existe en TPuy
- **Solución**: Terminar métodos con solo `RETURN`

### 4. Control de permisos en UI
- **Problema**: Usuarios sin permisos podían acceder a botones de guardado
- **Solución**: Validar con `hb_IsObject()` + `IsAdmin()/IsDevel()` antes de cada operación

## Lecciones Aprendidas

1. **Siempre validar objetos antes de acceder**: Usar `hb_IsObject()`, `hb_IsHash()`, `hb_HHasKey()` antes de acceder a propiedades
2. **No asumir que funciones existen**: En Harbour solo existen las funciones documentadas en TPuy
3. **Mover validación después de definiciones**: En UI, el código de deshabilitar debe estar DESPUÉS de las definiciones de objetos
4. **Perfiles de usuario**:
   - Admin/Devel: acceso total
   - Propio usuario: puede cambiar contraseña (no auditor)
   - Auditor/Demo: sin acceso

## Archivos Modificados

| Archivo | Cambio |
|---------|--------|
| `tusuario.prg` | Perfil con valores por defecto |
| `guser.xbs` | Métodos con validaciones defensivas |
| `netio_login.xbs` | Validación de hUserData |
| `mae_usuarios.xbs` | Control de permisos por perfil |
| `mae_itf.xbs` | Solo admin/devel pueden crear IGTF |
| `vta_facturas.xbs` | Botón guardar según perfil |
| `vta_notas_cd.xbs` | Botón guardar según perfil |
| `vta_pedidos.xbs` | Botón guardar según perfil |
| `vta_notas_entrega.xbs` | Botón guardar según perfil |
| `vta_clientes.xbs` | Botón guardar según perfil |

## Recomendaciones

1. **Antes de commit**: Verificar que `hb_IsObject()` y `hb_HHasKey()` existan en TPuy
2. **Validación de UI**: Siempre mover validaciones DESPUÉS de las definiciones de objetos
3. **Perfiles**: Documentar claramente los permisos de cada perfil en AGENTS.md