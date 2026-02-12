# Configuración de Cambios - Fiscalius Client

## 2026-02-07 - Preparación para Gestión Segura de API Keys

### Cambios Realizados:
1. **Archivo .env.example**: Plantilla para futuras variables de entorno
   - BIG_PICKLE_API_KEY: Configuración para API de Big Pickle
   - Notas sobre uso futuro y compatibilidad actual
   - Instrucciones de seguridad incluidas

2. **Archivo .gitignore**: 
   - Excluye archivos .env y credenciales sensibles
   - Filtra archivos temporales y logs con datos sensibles
   - Previene commits accidentales de información confidencial

### Estado Actual:
- **Configuración JSON existente**: Sin cambios, mantiene compatibilidad
- **Base de datos**: PostgreSQL (fiscalius/postgres/tgtkpower)
- **Esquemas**: orseit, public, tpy_global

### Próximos Pasos (cuando se apruebe):
1. Copiar `.env.example` a `.env`
2. Reemplazar `your_actual_big_pickle_api_key_here` con la API key real
3. Implementar lectura de variables en el código TPuy
4. Coordinar migración con Riztan Gutierrez

### Comunicación:
- Todos los cambios documentados para coordinación con riztan@gmail.com
- Cambios preparados para implementación gradual

---

## 2026-02-12 - Fix de Pérdida de Decimales en Impuesto al PVP

### Problema Resuelto:
1. **mae_inventario_edit.xbs**: El campo `imp_pvp` solo almacenaba la parte entera al guardar productos
   - **Causa raíz**: Uso de `VAL()` para conversión de string a número, que puede perder decimales
   - **Síntoma**: Valor 12.34 se guardaba como 12

### Cambios Realizados:
1. **Líneas 183-187**: Reemplazado conversión problemática
   ```xbase
   // ANTES (problemático):
   FOR EACH cCol IN {"precio_unitario","stock_total","stock_minimo","imp_pvp","codigo_barras"}
      if VALTYPE( hData[ cCol ] )="C"
         hData[ cCol ] := VAL( ::hData[cCol] )  // ← Pérdida de decimales
      endif
   NEXT
   
   // DESPUÉS (corregido):
   FOR EACH cCol IN {"precio_unitario","stock_total","stock_minimo","imp_pvp","codigo_barras"}
      if VALTYPE( hData[ cCol ] )="C"
         if cCol == "codigo_barras"
            hData[ cCol ] := hData[ cCol ]   // Mantener como string
         else
            hData[ cCol ] := ToNum( hData[ cCol ] )   // ← Usa ToNum() para decimales
         endif
      endif
   NEXT
   ```

2. **Línea 234**: Corregida conversión en modelo para display
   ```xbase
   // ANTES:
   hb_ntos(hData["imp_pvp"])   // Convertía 12.34 a "12"
   
   // DESPUÉS:
   TRANSFORM( hData["imp_pvp"], P_92 )   // Mantiene 12.34
   ```

### Impacto:
- ✅ **Corrige** almacenamiento correcto de decimales en `imp_pvp`
- ✅ **Mantiene** compatibilidad con `precio_unitario` y otros campos
- ✅ **Mejora** precisión en cálculos de impuestos

### Nota Importante:
- Se identificaron problemas similares en otros scripts (`vta_facturas.xbs`, `vta_notas_cd.xbs`)
- Por decisión del usuario, solo se aplica fix en `mae_inventario_edit.xbs` por ahora
- Los otros scripts deberán ser corregidos en futuras actualizaciones

---

## Configuración del Servidor
Ver `../fiscalius-server/CHANGELOG.md` para cambios en el lado del servidor