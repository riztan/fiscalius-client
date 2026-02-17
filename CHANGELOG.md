# Configuración de Cambios - Fiscalius Client

## 2026-02-17 - Migración fin_recibo.xbs a CxCxFlujo()

### Cambios Realizados:
1. **Migración a CxCxFlujo()**: Cambia de `CuentasPorCobrar()` al nuevo método
   - `__fr_LoadDocs()` ahora usa `::rFinanzas:CxCxFlujo()` en lugar de `CuentasPorCobrar()`

2. **Visualización de flujos**: Muestra los flujos del documento en textview
   - `__fr_textview()` ahora parsea JSON de columna 28 (derivados)
   - Genera texto con tipo, monto y número de cada flujo

3. **Mejora parsing JSON**: Validación de tipos al procesar derivados
   - Verifica `VALTYPE()` antes de convertir valores
   - Maneja tanto strings como números

4. **Actualización de columnas**: Agrega campos de conversión monetaria
   - Tasa Actual, Monto Convertido (USD), Saldo Convertido (USD)

### Impacto:
- Recibo de cobranza ahora usa el nuevo método optimizado
- Visualización completa de flujos (IVA, retenciones, etc.)

## 2026-02-16 - Formulario Medios de Pago (Draiver Rojas)

### Cambios Realizados:
- Agrega formulario de gestión de medios de pago
- Nuevos scripts relacionados con mae_monederos

---

## 2026-02-17 - Bug Fix: Cálculo de Impuesto PVP × Cantidad

### Problema Resuelto:
1. **vta_facturas.xbs línea 928**: El cálculo de `imp_pvp` no multiplicaba por cantidad
   - **Causa raíz**: Código comentaba la multiplicación y usaba solo el valor unitario
   - **Síntoma**: Factura con 2 unidades de producto con imp_pvp=4.34 mostraba otros_imp=4.34 en lugar de 8.68

### Cambios Realizados:
1. **Línea 928**: Corregido el cálculo
   ```xbase
   // ANTES (incorrecto):
   nImpPvp := ROUND( ToNum( oTView:GetAutoValue( 21, aIter ) ), 2 ) // NO multiplicaba por cantidad
   
   // DESPUÉS (correcto):
   nImpPvp := ROUND( ToNum( oTView:GetAutoValue( 21, aIter ) ) * ToNum( oTView:GetAutoValue( __CANTIDAD, aIter ) ), 2 )
   ```

### Impacto:
- Factura #7 ahora muestra correctamente: 116 + 18.56 (IVA) + 8.68 (imp_pvp × 2) = 143.24 Bs
- Coherencia verificada: BD = API = Cliente

### Revisión de Coherencia Realizada:
- **Servidor (fiscalius-server)**: CxCxFlujo() sin valores hardcoded
- **Base de datos**: otros_imp y monto_total corregidos para factura #7
- **Cliente**: Bug de cálculo × cantidad corregido

---

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