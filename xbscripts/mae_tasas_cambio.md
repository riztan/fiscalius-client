# mae_tasas_cambio.xbs - Tasas de Cambio

## Descripción
Formulario para gestionar Tasas de Cambio históricas en Fiscalius.

**Estado**: ✅ Implementado (01/04/2026)

## Tabla
- **Schema**: `tpy_global`
- **Tabla**: `mae_monedass_transacciones`

---

## Estructura del Formulario

### Procedimiento Principal: `mae_tasas_cambio()`
- Lista todas las tasas de cambio disponibles
- Permite agregar nuevas tasas

### Columnas del ListBox
| # | Campo | Descripción |
|---|-------|-------------|
| 1 | ID | Identificador (oculto) |
| 2 | Moneda | Código de moneda |
| 3 | Nombre | Nombre de la moneda |
| 4 | Fecha | Fecha de la tasa |
| 5 | Valor (Bs) | Valor en bolívares |
| 6 | Moneda Base | Moneda base |
| 7 | Nombre Base | Nombre de la moneda base |

---

## Funcionalidad

### Agregar Nueva Tasa
- **Botón**: Nuevo (bNew)
- **Función**: `tasacamb_edit()`
- Permite crear nuevas tasas de cambio

### Modificar Tasa
- **Estado**: ComENTADO (actualmente no permitido)

### Eliminar Tasa
- **Botón**: Eliminar (bDel)
- **Función**: `tasacamb_delete()`
- Permite eliminar la última tasa de cambio

---

## Función: tasacamb_delete()

### Descripción
Elimina una tasa de cambio del sistema.

### Parámetros
```xbscript
procedure tasacamb_delete( oFormParent, oAliWnd, oModel )
```

### Lógica
1. **Obtiene datos** del registro seleccionado (ID, Moneda, Fecha, Valor)
2. **Confirma eliminación** con `MsgYesNo()`
3. **Crea instancia** de la clase `TTASACAMBIO` para eliminar
4. **Ejecuta Delete** en el servidor via `rTasa:Delete()`
5. **Elimina del modelo** con `oModel:Delete()`
6. **Muestra mensaje** de éxito o error

### Validaciones
- Solo permite eliminar la última tasa de cambio
- Validación en el servidor (clase `empresa.prg`)

### Código
```xbscript
procedure tasacamb_delete( oFormParent, oAliWnd, oModel )
   local nId, nValor, cFecha, cMoneda, rTasa
   
   nId     := oModel:GetCol( 1 )
   cMoneda := oModel:GetCol( 2 )
   cFecha  := oModel:GetCol( 4 )
   nValor  := oModel:GetCol( 5 )
   
   if !MsgYesNo( "¿Está seguro de eliminar esta tasa de cambio?" + hb_eol() + ;
                "Moneda: " + cMoneda + hb_eol() + ;
                "Fecha: " + cFecha + hb_eol() + ;
                "Valor: " + TRANSFORM(nValor, P_92), ;
                "Confirmar eliminación" )
      return .F.
   endif
   
   rTasa := oFormParent:oGEmpresa:GetTasaCambio( { "id" => nId } )
   
   if rTasa:Delete( nId, cFecha )
      oModel:Delete()
      MsgInfo( "Tasa de cambio eliminada correctamente.", "Éxito" )
      return .T.
   else
      MsgAlert( "No se pudo eliminar la tasa de cambio.", "Error" )
      return .F.
   endif

return .F.
```

---

## Clase del Servidor

### TTASACAMBIO (empresa.prg)
- **METHOD GetTasaCambio()**: Obtiene instancia de tasas
- **METHOD Delete(id,cFecha)**: Elimina tasa de cambio

### Validaciones del Servidor
1. Verifica formato de fecha
2. Verifica que sea la última tasa
3. Ejecuta DELETE en BD

---

## Auditoría
Esta tabla tiene trigger de auditoría:
- **Trigger**: `trg_audit_mae_monedass_transacciones`
- **Tabla**: `orseit.auditoria_log`
- **Documentación**: `docs/auditoria_mae_monedass_transacciones.md`

---

## Archivos Relacionados
- **Cliente**: `xbscripts/mae_tasas_cambio.xbs`
- **UI**: `resources/mae_tasas_cambio.ui`
- **Servidor**: `apps/fiscalius/empresa.prg` (TTASACAMBIO)
- **Documentación**: `docs/auditoria_mae_monedass_transacciones.md`
