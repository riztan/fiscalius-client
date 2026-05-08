# Instrucciones para GitHub Copilot - Fiscalius-Client

**Proyecto**: Cliente TPuy/xHarbour + GTK  
**Directorio**: `/home/riztan/git/fiscalius-client`

---

## 📋 PLANIFICACIÓN: ANALIZA LO EXISTENTE PRIMERO

Antes de escribir código nuevo:
- Busca cómo se resolvió algo similar anteriormente
- Prioriza patrones ya establecidos en el proyecto
- Si necesitas cambiar una estructura, pregunta antes de actuar

---

## ⚠️ REGLA CRÍTICA: NO MODIFICAR ARCHIVOS SIN PERMISO

- NO modificar archivos que funcionan correctamente
- Si el sistema está operativo, NO tocar el código
- Primero PREGUNTAR al usuario antes de cualquier modificación
- Solo modificar si el usuario SOLICITA un cambio específico

---

## ⏱️ TIEMPO LÍMITE: PIDE AYUDA A TIEMPO

**Si en ~1 minuto no encuentras lo que buscas, DETENTE y pregunta.**

No deduzcas más de 1 minuto buscando el origen de algo. Es mejor pedir ayuda que asumir incorrectamente.

---

## ⚠️ IMPORTANTE: NO INVENTAR CÓDIGO

Antes de sugerir procedimientos o funciones, verifica que existen en el proyecto.

---

## 🔍 PROCEDIMIENTOS DISPONIBLES

### Módulos principales:
- `vta_facturas(oFormParent, cTipoDocumento)`
- `vta_notas_cd(oFormParent, cTipo)`
- `vta_notas_entrega(oFormParent)`
- `fin_recibo(oFormParent)`
- `fin_cajas(oFormParent)`
- `mae_inventario(oFormParent)`

### Objetos remotos del servidor:
- `rEmpresa:SetModulo("VENTAS")` → rVentas
- `rEmpresa:SetModulo("FINANZAS")` → rFinanzas
- `rVentas:Clientes(hParams)` → cursor
- `rVentas:Facturas(cId)` → objeto VFACTURA
- `rFinanzas:CxCxFlujo(hParams)` → cursor CxC
- `rFinanzas:Cobranza(cId)` → objeto TFRECIBO
- `rFinanzas:Cajas()` → cursor

---

## ✅ PATRONES SEGUROS

### Conversión numérica (SIEMPRE ToNum):
```xbase
nValor := ToNum( cCampo )  // ✅ CORRECTO
nValor := VAL( cCampo )    // ❌ INCORRECTO
```

### Verificar conexión:
```xbase
IF !oTPuy:RunXBS("netio_check")
   MsgStop("Sin conexión")
   RETURN
ENDIF
```

### Cargar datos a ListBox:
```xbase
oDatos := rEmpresa:GetData()
oModel:oTreeView:ClearModel(.T.)
FOR EACH aLine IN oDatos:GetData()
   APPEND LIST_STORE oModel:oLbx ITER aIter VALUES aLine[1], aLine[2], aLine[3]
NEXT
```

---

## 🚫 NO SUGERIR

- Usar VAL() en lugar de ToNum()
- Operaciones dentro de VALUES de LIST_STORE
- Acceso a propiedades de objetos nulos
- Procedimientos que no existen en xbscripts/

---

## 📖 Referencia

Ver `docs/directrices-ia.md` y `.agent/skills/fiscalius_client/SKILL.md`