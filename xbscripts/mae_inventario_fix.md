# Fixes de mae_inventario_edit.xbs

Este documento registra los bugs y sus soluciones implementadas en el módulo de inventario.

---

## 🐛 Fix #1: Pérdida de Decimales en imp_pvp

## 🐛 **Problema Identificado**

Al guardar un producto en el inventario, el campo `imp_pvp` (impuesto al precio de venta) solo almacenaba la parte entera, perdiendo los decimales.

**Síntoma:** Valor `12.34` se guardaba como `12`

---

## 🔍 **Análisis del Problema**

### **Ubicación del Bug**
- **Archivo**: `mae_inventario_edit.xbs`
- **Líneas afectadas**: 183-187 y 234

### **Causa Raíz**

#### **1. Problema de Conversión (Líneas 183-187)**
```xbase
FOR EACH cCol IN {"precio_unitario","stock_total","stock_minimo","imp_pvp","codigo_barras"}
   if VALTYPE( hData[ cCol ] )="C"
      hData[ cCol ] := VAL( ::hData[cCol] )  // ← PÉRDIDA DE DECIMALES
   endif
NEXT
```

**Problema:** `VAL()` puede perder decimales en ciertas condiciones regionales.

#### **2. Problema de Display (Línea 234)**
```xbase
hb_ntos(hData["imp_pvp"])   // ← CONVIERTE 12.34 A "12"
```

**Problema:** `hb_ntos()` convierte números a strings sin decimales.

---

## 🛠️ **Solución Implementada**

### **1. Corrección de Conversión (Líneas 183-187)**

#### **ANTES (Problemático):**
```xbase
FOR EACH cCol IN {"precio_unitario","stock_total","stock_minimo","imp_pvp","codigo_barras"}
   if VALTYPE( hData[ cCol ] )="C"
      hData[ cCol ] := VAL( ::hData[cCol] )  // ← Pérdida de decimales
   endif
NEXT
```

#### **DESPUÉS (Corregido):**
```xbase
FOR EACH cCol IN {"precio_unitario","stock_total","stock_minimo","imp_pvp","codigo_barras"}
   if VALTYPE( hData[ cCol ] )="C"
      if cCol == "codigo_barras"
         hData[ cCol ] := hData[ cCol ]   // Mantener como string
      else
         hData[ cCol ] := ToNum( hData[ cCol ] )   // ← ToNum() maneja decimales
      endif
   endif
NEXT
```

### **2. Corrección de Display (Línea 234)**

#### **ANTES:**
```xbase
hb_ntos(hData["imp_pvp"])   // Convertía 12.34 a "12"
```

#### **DESPUÉS:**
```xbase
TRANSFORM( hData["imp_pvp"], P_92 )   // Mantiene formato con 2 decimales
```

---

## 📋 **¿Por qué ToNum() es mejor que VAL()?**

| Función | Ventajas | Desventajas |
|---------|----------|------------|
| **`VAL()`** | Función estándar Harbour | Puede perder decimales dependiendo de configuración regional |
| **`ToNum()`** | Maneja correctamente configuraciones regionales y decimales | Requiere importar librerías adicionales (ya disponibles en TPuy) |

---

## ✅ **Impacto de la Solución**

- ✅ **Corrige** almacenamiento correcto de decimales en `imp_pvp`
- ✅ **Mantiene** compatibilidad con `precio_unitario` y otros campos
- ✅ **Preserva** comportamiento para campos no numéricos (`codigo_barras`)
- ✅ **Mejora** precisión en cálculos fiscales

---

## 🧪 **Pruebas Recomendadas**

1. **Guardar nuevo producto** con `imp_pvp = 12.34` → verificar se guarda como `12.34`
2. **Guardar producto** con `imp_pvp = 100.00` → verificar se guarda correctamente
3. **Editar producto existente** → verificar decimales se mantienen
4. **Validar `precio_unitario`** → verificar que también mantenga decimales
5. **Validar `codigo_barras`** → verificar se mantiene como string

---

## ⚠️ **Problemas Similares Identificados**

### **Scripts con Mismo Patrón (Sin Corregir por Ahora)**

#### **🔴 Crítico**
1. **`vta_facturas.xbs`** (líneas 1224-1240)
   - Afecta: `precio_unitario`, `subtotal`, `iva_monto`, `total`, `imp_pvp`
   - Impacto: **ALTO** - Cálculos de facturación

2. **`vta_notas_cd.xbs`** (líneas 1392-1400)
   - Afecta: `precio_unitario`, `subtotal`, `total`
   - Impacto: **ALTO** - Notas de crédito/débito

#### **🟡 Medio**
3. **`vta_facturas.xbs`** (múltiples líneas con `hb_ntos()`)

### **Patrón a Buscar**
```xbase
// 🔴 SOSPECHOSO:
hData["campo_decimal"] := VAL( string_value )
oModel:Set( aIter, nCol, hb_ntos( decimal_value ) )

// ✅ CORRECTO:
hData["campo_decimal"] := ToNum( string_value )
oModel:Set( aIter, nCol, TRANSFORM( decimal_value, P_92 ) )
```

---

## 📚 **Referencias Relacionadas**

- **Fix implementado**: `mae_inventario_edit.xbs:183-187, 234`
- **Documentación general**: `../docs/bugs-fixes.md`
- **Changelog**: `../CHANGELOG.md`
- **Manual programador**: `../docs/manual-programador.md`

---

## 🔄 **Historial del Fix**

- **2026-02-12**: Identificación y corrección del problema
- **Responsable**: Asistente IA con revisión y aprobación del usuario
- **Aprobado por**: Usuario (decisión de aplicar solo en mae_inventario_edit.xbs)

---

*Este documento debe mantenerse actualizado con cualquier cambio relacionado con el manejo de decimales en el módulo de inventario.*

---

## 🐛 Fix #2: Campo "Incluye IVA" No se Guardaba

### **Problema Identificado**

Al marcar o desmarcar el checkbox "Incluye IVA" en el formulario de productos, el cambio no se reflejaba en la base de datos.

**Síntoma:** El campo `incluye_iva` siempre mantenía su valor original después de guardar

---

### **Análisis del Problema**

#### **Ubicación del Bug**
- **Archivo**: `mae_inventario_edit.xbs`
- **Línea afectada**: 176-187 (procedimiento `__MaeInvSave`)

#### **Causa Raíz**
El campo `incluye_iva` **no se procesaba** antes de guardar. La conversión solo afectaba a campos numéricos:

```xbase
// Código original - solo procesaba estos campos:
FOR EACH cCol IN {"precio_unitario","stock_total","stock_minimo","imp_pvp","codigo_barras"}
   // ... conversión
NEXT

// ❌ incluye_iva NO estaba en la lista
```

Además, había código comentado que intentaba manejar este campo:

```xbase
// Código comentado que no se ejecutaba:
//   if VALTYPE( ::hData["incluye_iva"] )="C"
//      hData["incluye_iva"] :=  iif( ALLTRIM(Upper(::hData["incluye_iva"]))="S",.T.,.F. )
//   endif
```

---

### **Solución Implementada**

#### **Verificar estructura de la tabla**
```sql
-- Tabla: orseit.mae_inventario
-- Campo: incluye_iva (boolean)
```

#### **Corrección implementada (Líneas 179-187)**

```xbase
// Convertir incluye_iva a boolean para PostgreSQL
if hb_hHasKey(hData, "incluye_iva")
   if VALTYPE(hData["incluye_iva"]) = "C"
      hData["incluye_iva"] := iif( ALLTRIM(Upper(hData["incluye_iva"]))="S", .T., .F. )
   elseif VALTYPE(hData["incluye_iva"]) = "N"
      hData["incluye_iva"] := iif( hData["incluye_iva"] = 1, .T., .F. )
   endif
endif
```

#### **Qué hace la solución:**
1. Verifica si la clave `incluye_iva` existe en el hash de datos
2. Si viene como **string** ("S" o "N"): convierte a boolean
3. Si viene como **número** (1 o 0): convierte a boolean
4. PostgreSQL recibe el tipo correcto (boolean)

---

### **Pruebas Recomendadas**

1. **Crear nuevo producto** con "Incluye IVA" marcado → verificar en BD
2. **Editar producto existente** y cambiar el checkbox → verificar cambio en BD
3. **Desmarcar "Incluye IVA"** → verificar que se guarde como `false`
4. **Verificar consulta SQL:**
   ```sql
   SELECT codigo_local, incluye_iva FROM orseit.mae_inventario;
   ```

---

### **Referencia Técnica**

- **Tabla**: `orseit.mae_inventario`
- **Campo**: `incluye_iva` (boolean, nullable)
- **Componente UI**: `CHECKBOX` en `mae_inventario.ui`

---

## 📚 **Referencias Relacionadas**

- **Fix #1 (imp_pvp)**: `mae_inventario_edit.xbs:183-187, 234`
- **Fix #2 (incluye_iva)**: `mae_inventario_edit.xbs:179-187`
- **Documentación general**: `../docs/bugs-fixes.md`
- **Changelog**: `../CHANGELOG.md`

---

## 🔄 **Historial de Fixes**

| # | Fecha | Problema | Estado |
|---|-------|----------|--------|
| 1 | 2026-02-12 | Pérdida de decimales en imp_pvp | ✅ Corregido |
| 2 | 2026-02-13 | Campo incluye_iva no se guardaba | ✅ Corregido |

- **Responsable**: Asistente IA con revisión y aprobación del usuario
- **Aprobado por**: Usuario

---

*Este documento debe mantenerse actualizado con cualquier cambio relacionado con el módulo de inventario.*