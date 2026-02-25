# Manual Técnico para Programadores - Fiscalius Client

## 🎯 **Resumen del Proyecto**

Fiscalius Client es un sistema de facturación electrónica desarrollado con **TPuy** (framework propio basado en Harbour + T-GTK), diseñado como cliente de la arquitectura 3-capas que interactúa con **fiscalius-server**.

---

## 🏗️ **Arquitectura del Sistema**

### **Estructura General**
```
┌─────────────────┐    NetIO/HTTP    ┌──────────────────┐    SQL    ┌─────────────┐
│  Cliente TPuy   │ ←──────────────→ │ Fiscalius-Server │ ←───────→ │ PostgreSQL  │
│  (UI + Val)     │                  │  (API + Negocio) │           │   (Datos)   │
└─────────────────┘                  └──────────────────┘           └─────────────┘
```

### **Capas del Sistema**
1. **Presentación (Cliente TPuy)**: Interfaz GTK+ + validaciones previas
2. **Negocio (fiscalius-server)**: API REST/NetIO + lógica de negocio  
3. **Datos (PostgreSQL)**: Motor de base de datos principal

---

## 🔧 **Framework TPuy**

### **Componentes Principales**
- **Base**: Harbour + T-GTK (https://github.com/FiveTechSoft/T-Gtk)
- **Framework**: TPuy (https://github.com/riztan/TPuy)
- **Repositorio**: `~/tpuy` (local), GitHub remoto
- **Versión**: Última disponible

### **Archivos de Configuración**
- **`tpy_xbs.ch`**: Definiciones para scripts TPuy (uso general)
- **`tpy_class.ch`**: Definiciones exclusivas para clases
- **`tpy_netio.ch`**: Gestión de comunicación remota
- **`connect.ch`**: Configuración de conexión al servidor

---

## 💾 **Sistema de Objetos**

### **Jerarquía de Clases**
```
Empresa (oGEmpresa)
├── Módulo Ventas (rVentas)
│   ├── Facturas (rFacturas)
│   ├── Clientes (rClientes)
│   └── Notas (rNotas)
├── Módulo Finanzas (rFinanzas)
│   ├── Recibos (rRecibos)
│   └── Cajas (rCajas)
└── Módulo Inventario (rInventario)
```

### **Objetos Principales**
- **`oTPuy`**: Objeto global público (única variable global)
- **`oForm`**: Instancia local de TPublic para variables del formulario
- **`SET PUBLIC oForm`**: Azúcar sintáctico para `oForm := TPublic():New()`

### **Sintaxis Especial**
```xbase
::cNombre := "Abraham"     // Equivalente a: oForm:cNombre := "Abraham"
::aDetal := {}             // Array local en oForm
```

---

## 🌐 **Sistema de Comunicación NetIO**

### **Arquitectura de Comunicación**
1. **Cliente TPuy** → Llama método remoto (`~object:msg()`)
2. **Servidor hb_api_service** → Procesa y devuelve JSON con query
3. **TPuy intercepta** → Convierte JSON a `tcursor` local vía `rclasses.prg`
4. **Programador trabaja** → Con objeto `tcursor` como si fuera local

### **Sintaxis de Comunicación**
```xbase
// Mensajes de objeto
~object:msg[(params)]       // → FromRemote("__object", object, #msg[, params])
~object:method(params)      // → FromRemote("__objmethod", object, #method[, params])

// Atajos
r:object:msg                 // → FromRemote("__objmethod", object, #msg)
~~object:msg[(params)]      // → FromRemote("__object", object, #msg[, params])
```

### **Configuración de Conexión**
- **Variables**: `NETSERVER`, `NETPORT`, `NETPASSWD` (en `connect.ch`)
- **Endpoint**: Puerto NetIO del servidor
- **Autenticación**: Manejo implícito a través de sesión

---

## 🎨 **Interfaz Gráfica GTK+**

### **Diseño con Glade**
- **Herramienta**: Glade v3.8 (incluido en TPuy)
- **Archivos**: `.ui` en carpeta `resources/`
- **Manipulación**: T-GTK + GTKBuilder

### **Componentes UI**
```xbase
// Cargar recursos
SET RESOURCES oRes FROM FILE oTPuy:cResources+"vta_facturas.ui"

// Definir ventana
DEFINE WINDOW ::oFacWnd TITLE "Fiscalius. Factura" SIZE 800,600

// Definir componentes
DEFINE BUTTON ::oBtnGuardar ACTION __VtaSave( oForm, oFormParent )
DEFINE ENTRY ::oEntCli VAR ::cCli ACTION oTPuy:RunXBS("vta_clientes")

// Modelo y Listbox
DEFINE MODEL oModClientes STRUCT rClientes:Struct() DATA rClientes:GetData()
DEFINE LISTBOX oLBoxCli MODEL oModClientes
```

### **Variables y Alcance**

- **`::variable`** → solo para **variables de instancia de clase** (en métodos de clase)
- **`local variable`** → en **procedures** (no tienen clase)
- **ERROR común**: usar `::` en procedures → compilación falla

### **ENTRY con UI**
- Requiere 2 variables:
  - `local oEntFecha` (el objeto Entry)
  - `local cEntFecha` (el valor/variable)
- Asignar valor inicial antes de DEFINE: `cEntFecha := ""`
- Definir con `VAR cEntFecha` y `ID` del .ui
```xbase
local oEntFecha, cEntFecha
cEntFecha := ""
DEFINE ENTRY TYPE DATE oEntFecha VAR cEntFecha CALENDAR ;
       ID "ent_fecha" RESOURCE oRes
```

### **BOX y Botones del UI**
```xbase
// BOX del UI
DEFINE BOX oBoxData ID "box_data" RESOURCE oRes EXPAND FILL

// Botones del UI
DEFINE BUTTON oBtnBuscar ID "btn_buscar" RESOURCE oRes
```

### **ListBox**
- Configuraciones (SetColVisible, SetColTitle) **antes** de `Active()`
```xbase
oLBox:lBar := .F.
oLBox:SetColVisible( 1, .F. )
oLBox:SetColTitle( 1, "Titulo" )
oLBox:Active()
```

### **Refrescar Modelo (Botón Buscar)**
Para refrescar datos en un ListBox:
```xbase
// 1. Obtener nuevos datos
oDatos := rEmpresa:Metodo( hParam )

// 2. Limpiar modelo existente
oModel:oTreeView:ClearModel(.t.)

// 3. Agregar datos al modelo
FOR EACH aLine IN oDatos:GetData()
   APPEND LIST_STORE oModel:oLbx ITER aIter ;
          VALUES aLine[1], aLine[2], aLine[3], aLine[4]
NEXT
```

### **Conversión de Fechas (Servidor)**
Manejar diferentes formatos de fecha string:
```xbase
// Convertir string a fecha
dFecha := CTOD(cFecha)
if empty(dFecha) .and. "-" $ cFecha
   dFecha := STOD( STRTRAN( cFecha, "-", "" ) )
endif
if empty(dFecha) .and. "/" $ cFecha
   if Len(cFecha) == 10
      dFecha := STOD( SubStr(cFecha,7,4) + SubStr(cFecha,4,2) + SubStr(cFecha,1,2) )
   endif
endif
if empty(dFecha) .and. Len(cFecha) == 8 .and. !("/" $ cFecha) .and. !("-" $ cFecha)
   dFecha := STOD( cFecha )
endif
```

### **Manejo de Eventos**
- **Sistema**: Basado en señales GTK estándar
- **Gestión**: T-GTK (gclass) - https://github.com/FiveTechSoft/T-Gtk/tree/master/src/gclass
- **Callbacks**: `ACTION function(params)` en componentes

---

## 📊 **Manejo de Datos**

### **Flujo de Datos**
```xbase
// 1. Obtener objeto remoto (tcursor local transparente)
rClientes := rVentas:Clientes(hParam)

// 2. Obtener estructura (metadata tipo DBF)
aStruct := rClientes:Struct()

// 3. Obtener datos puros (arreglo)
aData := rClientes:GetData()

// 4. Usar en UI
DEFINE MODEL oModClientes STRUCT aStruct DATA aData
```

### **Métodos Principales**
- **`Struct()`**: Metadata compatible con estructura DBF
- **`GetData()`**: Arreglo puro con solo los datos
- **Query remoto**: Retorna JSON → `tcursor` local

### **Listbox y Grids**
```xbase
oLBoxCli:lBar := .f.                    // Sin barra de herramientas
oLBoxCli:bEdit := {|oModel| action() }  // Callback de edición
oLBoxCli:SetColVisible(1, .f.)          // Ocultar columna
oLBoxCli:SetColTitle(2, "Código")       // Cambiar título
```

---

## 💻 **Programación xHarbour**

### **Tipos de Datos y Validación**
```xbase
// Validaciones estándar Harbour
hb_isObject(oX)      // ¿Es objeto?
hb_isNIL(x)         // ¿Es nulo?
hb_isArray(a)       // ¿Es array?

// Arrays heterogéneos
aDetal := {"string", 123, .t., oObject}
```

### **Funciones vs Procedimientos**
```xbase
// PROCEDURE - No retorna valor
PROCEDURE SaveInvoice(oForm)
   // Lógica de guardado
RETURN

// FUNCTION - Sí retorna valor  
FUNCTION CalculateIVA(nAmount)
   RETURN nAmount * 0.16
```

### **Variables y Alcance**
```xbase
// Global única
oTPuy  // Accesible desde cualquier script

// Local de formulario
SET PUBLIC oForm  // oForm := TPublic():New()
::cNombre := "test"  // Vive con oForm

// Local de función
LOCAL cNombre, nEdad, aDatos  // Solo en esta función
```

---

## 📁 **Estructura de Proyecto**

### **Directorios Principales**
```
fiscalius-client/
├── xbscripts/          # Scripts XBScript (lógica del negocio)
│   ├── vta_*.xbs       # Ventas
│   ├── mae_*.xbs       # Maestros
│   ├── fin_*.xbs       # Finanzas
│   └── netio_*.xbs     # Conexión
├── resources/          # Interfaces GTK (.ui)
├── images/            # Recursos gráficos
├── include/           # Librerías y headers
└── docs/              # Documentación
```

### **Convenciones de Nomenclatura**
- **Prefijos de módulo**: `vta_` (ventas), `mae_` (maestros), `fin_` (finanzas)
- **Funciones internas**: `__FunctionName()` (privadas del módulo)
- **Variables**: Prefijo húngaro (`c`=char, `n`=numeric, `a`=array, `o`=object, `l`=logical)

---

## 🛠️ **Flujo de Desarrollo**

### **Inicio del Sistema**
1. **`begin.xbs`**: Inicializa TPuy, configura tema y temporizadores
2. **Timer NetIO**: `__NetIOUpdate()` → verifica conexión periódicamente
3. **`netio_check.xbs`**: Conecta al servidor y verifica disponibilidad
4. **`__Login()`**: Ventana de autenticación via `netio_login.xbs`
5. **Menú principal**: Sistema listo para operación

### **Patrones de Programación**
```xbase
// Patrón estándar de módulo
procedure vta_facturas( oFormParent )
   // 1. Verificar conexión
   if !oTpuy:RunXBS("netio_check")
      MsgStop("Problemas para continuar...")
   endif
   
   // 2. Obtener objetos remotos
   ::rVentas := oFormParent:oGEmpresa:SetModulo("Ventas")
   ::rFacturas := ::rVentas:Facturas()
   
   // 3. Cargar interfaz
   SET RESOURCES ::oRes FROM FILE oTPuy:cResources+"vta_facturas.ui"
   
   // 4. Definir componentes y callbacks
   // ...
RETURN
```

---

## 🔍 **Depuración y Desarrollo**

### **Herramientas de Depuración**
- **Principal**: `View(variable)` - inspecciona cualquier variable
- **Logs**: Archivos tradicionales (`comp.log`, `error_.log`)
- **Editor**: gvim (o editor de texto preferido)
- **Sin IDE especial**: Enfoque minimalista tradicional

### **Técnicas de Depuración**
```xbase
// Ver contenido de variables
View( oForm )
View( aDatos )
View( ::cVariable )

// Ver estructura de objetos
View( rFacturas:Struct() )
View( rFacturas:GetData() )
```

---

## 🌍 **Integraciones Externas**

### **Estado Actual**
- **SENIAT**: Pendiente de APIs oficiales
- **Impresión fiscal**: Disponible externamente, no integrado aún
- **Tasas de cambio BCV**: Gestión via servidor (no en cliente)
- **Reportes**: Excel con `hbxlsxwriter.ch`, sistema programático

### **API Server**
- **Endpoint**: https://localhost:8001/tpy
- **Documentación**: `~/git/fiscalius-server/docs/API_DOCUMENTACION.md`
- **Interfaz web**: https://localhost:8001/html

---

## 📚 **Referencias y Recursos**

### **Repositorios Clave**
- **TPuy Framework**: https://github.com/riztan/TPuy
- **T-GTK**: https://github.com/FiveTechSoft/T-Gtk
- **Harbour Core**: https://github.com/harbour/core
- **NetIO**: https://github.com/harbour/core/tree/master/contrib/hbnetio
- **Fiscalius-Server**: `~/git/fiscalius-server/`

### **Documentación Local**
- **API Server**: `~/git/fiscalius-server/docs/API_DOCUMENTACION.md`
- **Manual Técnico**: `~/git/fiscalius-server/docs/manual_tecnico.md`
- **Respuestas Técnicas**: `docs/respuestas-tecnicas.md` (este proyecto)

---

## 🚀 **Buenas Prácticas**

### **Programación**
- Usar `SET PUBLIC` para crear instancias TPublic
- Seguir convenciones de prefijos (`vta_`, `mae_`, `fin_`)
- Marcar funciones internas con `__` 
- Validar con `hb_is*()` antes de operar objetos
- Usar `View()` para depuración en tiempo real

### **Arquitectura**
- Cliente solo maneja UI y validaciones previas
- Toda lógica de negocio va al servidor
- Usar NetIO para comunicación cliente-servidor
- Mantener separación clara de responsabilidades

### **Mantenimiento**
- Documentar cambios en código
- Seguir patrones existentes
- Probar conexión NetIO antes de operaciones
- Manejar gracefully fallas de conexión

---

## 🎯 **Próximos Pasos para Nuevos Desarrolladores**

1. **Configurar entorno**: Instalar TPuy y T-GTK
2. **Estudiar arquitectura**: Entender flujo cliente-servidor
3. **Revisar ejemplos**: Analizar scripts existentes (`vta_facturas.xbs`)
4. **Probar conexión**: Verificar `netio_check.xbs` funciona
5. **Crear módulo simple**: Basado en patrones existentes
6. **Documentar**: Agregar comentarios y seguir convenciones

---

**Este manual debe complementarse con el análisis de código fuente específico y la revisión de los scripts existentes en el proyecto.**