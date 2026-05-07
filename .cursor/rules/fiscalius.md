# Reglas Fiscalius-Client - Cursor IDE

**Proyecto**: Cliente de escritorio TPuy/xHarbour + GTK  
**Directorio**: `/home/riztan/git/fiscalius-client`

---

## 📋 REGLA DE PLANIFICACIÓN: PRIMERO PLAN, DESPUÉS CÓDIGO

**Antes de escribir código nuevo, analiza lo existente.**

- La mayoría de funcionalidades nuevas se basan en patrones ya implementados
- **PRIORIZA** buscar cómo se hizo algo similar antes de inventar
- Si necesitas romper una estructura establecida, **PREGUNTA PRIMERO**
- Discute el enfoque, planifica, y solo luego aplica lo acordado

> Ejemplo: "Veo que el módulo de facturas usa este patrón para el TreeView. ¿Aplico el mismo o hay otra forma preferida?"

---

## ⏱️ REGLA DE TIEMPO: PIDE AYUDA A TIEMPO

**Si en ~1 minuto no encuentras la información que necesitas, DETENTE y pregunta al usuario.**

- **NO** deduzcas más de 1 minuto buscando el origen de algo
- **NO** sigas buceando en código endlessly para "comprender mejor"
- **SIEMPRE** prioriza solicitar ayuda cuando el tiempo límite se alcance
- Es mejor pedir ayuda que generar código incorrecto por asumir

> Ejemplo: "No encontré el procedimiento X en los xbscripts/. ¿Puedes indicarme dónde está o si debo usar otro?"

---

## 🚨 REGLA CRÍTICA: NO INVENTAR PROCEDIMIENTOS

**NUNCA** crear llamadas a procedimientos o funciones sin verificar que existen. El cliente se comunica con el servidor vía NetIO.

---

## 📋 PROCEDIMIENTOS Y FUNCIONES VERIFICADAS

### Conexión al Servidor
- `oTPuy:RunXBS("netio_check")` → Verifica conexión
- `oFormParent:oGEmpresa:SetModulo("VENTAS")` → Instancia módulo
- `oFormParent:oGEmpresa:SetModulo("FINANZAS")` → Módulo finanzas

### Objetos Remotos (del servidor)
```
rVentas:Clientes(hParams)     → Cursor clientes
rVentas:Facturas(cId)         → Instancia VFACTURA
rVentas:Documento(cTipo, cNumero) → Crea documento
rFinanzas:CxCxFlujo(hParams)  → CxC con flujos
rFinanzas:Cobranza(cId)       → Instancia TFRECIBO
rFinanzas:Cajas()             → Lista cajas
rFinanzas:MediosPago()        → Lista medios pago
rInventario:Productos(hParams) → Lista productos
```

### Métodos de Cursor Remoto
- `rCursor:GetData()` → Array de datos
- `rCursor:Struct()` → Estructura de columnas
- `rCursor:Eof()` → Fin de cursor
- `rCursor:FieldGet(n)` → Valor campo n

### Métodos de Objeto Remoto
- `rObjeto:Help()` → Lista métodos disponibles
- `rObjeto:Data()` → Hash con datos
- `rObjeto:SetData(hData)` → Actualiza datos
- `rObjeto:Save()` → Guarda en servidor
- `rObjeto:Delete()` → Elimina en servidor
- `rObjeto:Detal()` → Detalles documento

---

## 📋 PROCEDIMIENTOS LOCALES DEL CLIENTE

### Procedures principales (xbscripts/)
- `vta_facturas(oFormParent, cTipoDocumento)` → Módulo facturas
- `vta_notas_cd(oFormParent, cTipo)` → Notas crédito/débito
- `vta_notas_entrega(oFormParent)` → Notas de entrega
- `vta_pedidos(oFormParent)` → Pedidos
- `vta_clientes(oFormParent)` → Gestión clientes
- `fin_recibo(oFormParent)` → Recibos de cobranza
- `fin_cajas(oFormParent)` → Cajas registradoras
- `mae_inventario(oFormParent)` → Catálogo productos
- `mae_monedas(oFormParent)` → Configuración monedas
- `mae_serie_fiscal(oFormParent)` → Series documentales

### Procedimientos internos (prefijo __)
- `__vtaFactura(oForm, oFormParent, oBoxFactura, oModel, lNew)` → Formulario factura
- `__fr_LoadDocs(oForm)` → Carga documentos en recibo
- `__ModuloSave(oForm)` → Guardar módulo
- `__ModuloDelete(oForm)` → Eliminar módulo
- `__LoadDetails(oForm)` → Carga detalles

### Funciones auxiliares
- `ToNum(cValor)` → Conversión numérica segura ⭐
- `View(xValue)` → Depuración (muestra variable)
- `MsgStop(cMsg)`, `MsgInfo(cMsg)`, `MsgYesNo(cMsg)` → Dialogs
- `hb_isObject(o)`, `hb_isNIL(x)`, `hb_isArray(a)` → Verificaciones

---

## ✅ PATRONES SEGUROS

### Llamada a servidor (NetIO):
```xbase
// Verificar conexión primero
IF !oTPuy:RunXBS("netio_check")
   MsgStop("Sin conexión al servidor")
   RETURN
ENDIF

// Obtener objeto remoto
::rFinanzas := oFormParent:oGEmpresa:SetModulo("Finanzas")
::rCobranza := ::rFinanzas:Cobranza('')

// Verificar respuesta
IF hb_isObject( ::rCobranza )
   hData := ::rCobranza:Data()
ENDIF
```

### Cargar datos a ListBox:
```xbase
// Obtener cursor remoto
oDatos := rEmpresa:Metodo( hParams )

// Limpiar modelo
oModel:oTreeView:ClearModel(.T.)

// Agregar datos
FOR EACH aLine IN oDatos:GetData()
   APPEND LIST_STORE oModel:oLbx ITER aIter VALUES aLine[1], aLine[2], aLine[3]
NEXT
```

### Obtener valor de celda:
```xbase
nId    := oTView:GetAutoValue( 1, aIter )   // Columna 1
cNombre:= oTView:GetAutoValue( 2, aIter )   // Columna 2
```

### Conversión numérica (SIEMPRE usar ToNum):
```xbase
// ✅ CORRECTO - mantiene decimales
nValor := ToNum( cValorCampo )

// ❌ INCORRECTO - VAL() puede perder decimales
nValor := VAL( cValorCampo )
```

### Verificar cursor antes de usar:
```xbase
IF hb_isObject( oDatos ) .AND. !oDatos:Eof()
   aData := oDatos:GetData()
ENDIF
```

---

## 🚫 PROHIBICIONES ABSOLUTAS

1. **NO usar VAL()** - usar siempre `ToNum()` para números con decimales
2. **NO acceder propiedades de objeto nil** - verificar con `hb_isNIL()` o `hb_isObject()`
3. **NO hacer operaciones en VALUES de LIST_STORE** - pasar valores ya calculados
4. **NO inventar procedimientos** - verificar con grep en xbscripts/
5. **NO acceder índices de array sin verificar** - usar `iif(LEN(a) >= n, a[n], valorDefault)`

---

## 📊 ESTRUCTURA DEL PROYECTO

```
xbscripts/
├── begin.xbs           # Entry point
├── netio_check.xbs     # Verifica conexión
├── netio_login.xbs     # Autenticación
├── menu.xbs            # Menú principal
├── vta_*.xbs           # Módulo ventas
├── mae_*.xbs           # Maestros
├── fin_*.xbs           # Finanzas
```

---

## 📚 DOCUMENTACIÓN

- `docs/directrices-ia.md` - Reglas del proyecto
- `docs/manual-programador.md` - Manual TPuy
- `AGENTS.md` - Guía de agentes
- `.agent/skills/fiscalius_client/SKILL.md` - Skill completo