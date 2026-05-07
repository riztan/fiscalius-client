---
name: fiscalius-client
description: Contexto completo, arquitectura, patrones de código y estado actual del cliente fiscalius (TPuy/xHarbour + GTK).
---

# Fiscalius Client — Skill Completo

## 🎯 Descripción del Proyecto

**Fiscalius-Client** es la capa de presentación del ERP Fiscal Fiscalius, desarrollada en **TPuy** (framework propio basado en Harbour + T-GTK). Es un cliente de escritorio GTK+ que se comunica con **fiscalius-server** vía protocolo **NetIO** (hb_netio). El cliente **no accede directamente a la base de datos** — toda la lógica de negocio reside en el servidor.

- **Directorio**: `/home/riztan/git/fiscalius-client`
- **Ejecución**: `tpuy` (desde el directorio del proyecto)
- **Framework**: TPuy
- **UI**: Glade v3.8 + T-GTK
- **Servidor**: `~/git/fiscalius-server` (puerto NetIO 2942)
- **Versión actual**: v2.0 (sincronizado con servidor)

---

## 🏗️ Arquitectura del Sistema

```
┌─────────────────────────┐   hb_netio TCP/IP   ┌───────────────────────┐   SQL   ┌────────────┐
│  Fiscalius Client       │ ←─────────────────→  │ Fiscalius Server      │ ←─────→ │ PostgreSQL │
│  (TPuy/xHarbour + GTK)  │                       │ (Harbour/TPuy)        │         │ 16         │
│  UI + Validaciones prev.│                       │ API + Lógica negocio  │         └────────────┘
└─────────────────────────┘                       └───────────────────────┘
```

### Comunicación NetIO
- **Función principal**: `FromRemote()` gestiona la comunicación
- **Sintaxis simplificada** (xtranslate en `tpy_netio.ch`):
  - `~objeto:msg(params)` → `FromRemote("__object", objeto, #msg, params)`
  - `r:objeto:msg` → `FromRemote("__objmethod", objeto, #msg)`
- **Respuesta**: JSON del servidor → objeto `tcursor` local (transparente para el programador)

---

## 📁 Estructura del Proyecto

```
fiscalius-client/
├── xbscripts/          # Lógica del negocio (.xbs)
│   ├── begin.xbs       # Punto de entrada principal
│   ├── netio_check.xbs # Verificación de conexión al servidor
│   ├── netio_login.xbs # Autenticación
│   ├── menu.xbs        # Menú principal
│   ├── vta_*.xbs       # Módulo Ventas
│   ├── mae_*.xbs       # Maestros (inventario, monedas, usuarios…)
│   └── fin_*.xbs       # Módulo Finanzas
├── resources/          # Interfaces GTK+ (.ui)
├── docs/               # Documentación
└── init.conf           # Configuración inicial
```

---

## 📋 Procedimientos y Funciones Locales

### Módulos principales (xbscripts/)
- `vta_facturas(oFormParent, cTipoDocumento)` → Gestión facturas
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

### Objetos remotos (del servidor)
- `oFormParent:oGEmpresa:SetModulo("VENTAS")` → rVentas
- `oFormParent:oGEmpresa:SetModulo("FINANZAS")` → rFinanzas
- `rVentas:Clientes(hParams)` → Cursor clientes
- `rVentas:Facturas(cId)` → Instancia VFACTURA
- `rVentas:Documento(cTipo, cNumero)` → Crea documento
- `rFinanzas:CxCxFlujo(hParams)` → CxC con flujos
- `rFinanzas:Cobranza(cId)` → Instancia TFRECIBO
- `rFinanzas:Cajas()` → Lista cajas
- `rFinanzas:MediosPago()` → Lista medios pago
- `rInventario:Productos(hParams)` → Lista productos

---

## 🔧 Patrones de Código

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
oDatos := rEmpresa:Metodo( hParams )
oModel:oTreeView:ClearModel(.T.)
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

### Variables locales (regla Harbour):
```xbase
// LOCAL debe declararse AL INICIO del procedimiento
PROCEDURE MiProc()
   LOCAL aLine, nLen, aVal
   FOR EACH aLine IN aData
      nLen := LEN( aLine )
      ...
   NEXT
RETURN
```

---

## ⏱️ REGLA DE TIEMPO: PIDE AYUDA A TIEMPO

**Si en ~1 minuto no encuentras la información que necesitas, DETENTE y pregunta al usuario.**

- **NO** deduzcas más de 1 minuto buscando el origen de algo
- **NO** sigas buceando en código endlessly para "comprender mejor"
- **SIEMPRE** prioriza solicitar ayuda cuando el tiempo límite se alcance
- Es mejor pedir ayuda que generar código incorrecto por asumir

> Ejemplo: "No encontré el procedimiento X en los xbscripts/. ¿Puedes indicarme dónde está?"

---

## 📋 REGLA DE PLANIFICACIÓN: PRIMERO PLAN, DESPUÉS CÓDIGO

**Antes de escribir código nuevo, analiza lo existente.**

- La mayoría de funcionalidades nuevas se basan en patrones ya implementados
- **PRIORIZA** buscar cómo se hizo algo similar antes de inventar
- Si necesitas romper una estructura establecida, **PREGUNTA PRIMERO**
- Discute el enfoque, planifica, y solo luego aplica lo acordado

---

## 🚫 REGLAS ABSOLUTAS (no negociables)

1. **NUNCA** modificar archivos `.xbs` o `.ui` sin permiso explícito del usuario
2. **NUNCA** hacer commits o pushes sin autorización
3. **NUNCA** usar valores hardcoded o inventar datos
4. **NUNCA** usar `VAL()` - usar siempre `ToNum()` para números con decimales
5. **SIEMPRE** verificar coherencia: BD = API = Cliente
6. **SIEMPRE** verificar objetos antes de acceder (`hb_isObject()`, `hb_isNIL()`)
7. **SIEMPRE** cerrar queries y limpiar recursos
8. **VERIFICAR** con el usuario antes de cualquier cambio estructural

### Checklist de seguridad:
- [ ] Leer docs/directrices-ia.md para reglas del proyecto
- [ ] Confirmar con el usuario que el cambio es necesario
- [ ] NO modificar nada hasta tener permiso explícito

---

## 📚 Documentación de Referencia

| Archivo | Contenido |
|---------|-----------|
| `docs/directrices-ia.md` | Reglas y directrices para IA (LEER SIEMPRE) |
| `docs/manual-programador.md` | Manual técnico completo del framework TPuy |
| `AGENTS.md` | Guía de agentes |
| `.agent/skills/fiscalius_client/SKILL.md` | Skill completo |

---

## 🏁 Checklist para Iniciar una Sesión

1. Escribir `/skill fiscalius_client` — Carga el skill
2. Leer `docs/directrices-ia.md` (reglas del proyecto)
3. Verificar que el servidor esté activo (en fiscalius-server)
4. **Preguntar al usuario: "¿Qué vamos a hacer hoy?"**
5. Confirmar con usuario qué tarea abordar hoy