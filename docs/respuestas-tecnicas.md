# Respuestas Técnicas - TPuy Framework

## Respuesta 1: TPuy Framework (versión, API, diferencias .ch)

### Versión Específica
- Framework TPuy desarrollado por el usuario con Harbour y T-GTK
- Versión: La última disponible
- TPuy local: `~/tpuy`
- Repositorio: https://github.com/riztan/TPuy
- T-GTK local: `~/t-gtk` (https://github.com/FiveTechSoft/T-Gtk)

### Documentación API
- La API está en el repositorio TPuy: https://github.com/riztan/TPuy
- Documentación disponible en el código fuente del framework

### Diferencias entre archivos .ch
- **`tpy_xbs.ch`**: Definiciones para scripts de TPuy (uso general)
- **`tpy_class.ch`**: Definiciones exclusivas para clases en scripts de TPuy
- `tpy_class.ch` se usa solo cuando se programa con clases

---

## Respuesta 2: Estructura de Objetos (:: sintaxis, jerarquía)

### Significado de `::` (doble dos puntos)
- Es un comodín definido en los archivos `.ch` 
- Asociado a la clase `tpublic` para manejo dinámico de variables
- Permite crear variables sin declaración previa
- `::cNombre := "Abraham"` es equivalente a `oForm:cNombre := "Abraham"`
- `oForm` es una instancia de `tpublic` que contiene un hash dinámico

### Jerarquía de Clases Principal
- **Empresa → Módulo → Documento**
- **Empresa**: `oGEmpresa` - contenedor principal
- **Módulos**: Ventas, Finanzas, Inventario (se obtienen con `SetModulo()`)
- **Documentos**: Facturas, Clientes, Recibos (objetos específicos de cada módulo)

### Creación de Instancias
- **Objeto Global**: `oTPuy` - siempre disponible públicamente desde cualquier script
- **Objeto Local**: `oForm` - instancia de `tpublic` para manejo de variables dinámicas
- **Módulos**: `rVentas := oGEmpresa:SetModulo("Ventas")`
- **Documentos**: `rFacturas := rVentas:Facturas()`

---

## Respuesta 3: Conexión NetIO (protocolo, seguridad)

### Protocolo NetIO
- Basado en **hbnetio** de Harbour Core: https://github.com/harbour/core/tree/master/contrib/hbnetio
- Protocolo TCP/IP estándar implementado por Harbour

### Comunicación en TPuy
- **Función principal**: `FromRemote()` gestiona toda la comunicación
- **Ubicación**: https://github.com/riztan/TPuy/blob/master/include/tpy_netio.ch
- **Sintaxis simplificada** (xtranslate):
  - `~object:msg[(params)]` → `FromRemote("__object", object, #msg[, params])`
  - `~object:method(params)` → `FromRemote("__objmethod", object, #method[, params])`
  - `r:object:msg` → `FromRemote("__objmethod", object, #msg)`
  - `~~object:msg[(params)]` → `FromRemote("__object", object, #msg[, params])`

### Estructura de Mensajes
- **Mensajes de objeto**: `"__object"` para obtener datos
- **Mensajes de método**: `"__objmethod"` para ejecutar métodos
- **Serialización**: `hb_deserialize()` para deserializar respuestas
- **Parámetros**: nombre del objeto y nombre del mensaje/método

### Autenticación y Seguridad
- Usa configuración de `connect.ch` (incluido automáticamente)
- Variables: `NETSERVER`, `NETPORT`, `NETPASSWD`
- Autenticación implícita a través de manejo de sesiones

### Gestión Avanzada de Mensajes (Servidor-Cliente)
- **Servidor**: `hb_api_service` (base de fiscalius-server): https://github.com/riztan/hb_api_service
- **Cliente TPuy**: Intercepta e interpreta mensajes del servidor
- **Procesamiento**: Crea objetos locales según tipo de mensaje recibido
- **Implementación**: https://github.com/riztan/TPuy/blob/master/src/rclasses.prg

### Transformación Transparente Remoto→Local
- **JSON → Objeto Local**: El servidor entrega JSON con estructura de datos tipo query
- **TCursor Creation**: TPuy genera un objeto `tcursor` para manipulación local
- **Transparencia Total**: El programador invoca método remoto y recibe objeto local
- **Ejemplo**: `rVentas:Clientes()` devuelve `tcursor` local trabajando con datos remotos

---

## Respuesta 4: Base de Datos (motor, esquemas, transacciones)

### Motores de Base de Datos Soportados por TPuy
- **MySQL**: Librerías integradas en TPuy
- **PostgreSQL**: Librerías integradas en TPuy
- **TPuy permite ambos** motores de forma nativa

### Arquitectura de Fiscalius Client (Capa de Presentación)
- **Sin acceso directo a base de datos**
- **Consume API de fiscalius-server** (~/git/fiscalius-server)
- **Comunicación vía API REST/NetIO**
- **Interfaz web disponible**: https://localhost:8001/html

### Servidor Fiscalius-Server (Capa de Negocio)
- **Ubicación**: ~/git/fiscalius-server y GitHub repositorio
- **Motor Principal**: PostgreSQL (según README del servidor)
- **Documentación API**: /home/riztan/git/fiscalius-server/docs/API_DOCUMENTACION.md
- **Endpoint Base**: https://localhost:8001/tpy
- **Puerto**: 8001 (HTTPS)

### Transacciones y Concurrencia
- **Manejado por el servidor**: fiscalius-server controla todo
- **Cliente dedicado a**: Interfaz de usuario y validaciones previas
- **Separación clara**: Cliente = UI/validación, Servidor = negocio/datos

### Flujo Arquitectónico
1. **Cliente TPuy** → Llama método remoto (`rVentas:Clientes()`)
2. **API Server** → Recibe petición HTTP/NetIO
3. **Servidor** → Procesa con PostgreSQL
4. **Respuesta** → JSON con estructura de query
5. **Cliente** → Recibe `tcursor` local transparente

---

## Respuesta 5: Modelos de Datos (Struct, GetData, métodos)

### 1. `rClientes:Struct()` - Estructura de Metadatos
- **Sí**, retorna metadata compatible con estructura DBF
- **Propósito**: Define columnas, tipos, longitudes
- **Uso**: Base para crear modelos de UI (`DEFINE MODEL oModClientes STRUCT rClientes:Struct()`)

### 2. `rVentas:Clientes()` - Consulta de Datos
- **Retorna**: Query de clientes (resultado de consulta SQL del servidor)
- **No es ORM**: El concepto de ORM no aplica directamente
- **Funcionamiento**: 
  1. Llamada remota vía NetIO
  2. Servidor ejecuta consulta SQL
  3. Retorna JSON con estructura query
  4. TPuy convierte a `tcursor` local

### 3. `GetData()` - Datos Puros
- **Retorna**: Arreglo con solo los datos correspondientes al `Struct()`
- **Sin metadata**: Solo valores, no estructura
- **Relación**: `Struct()` = estructura, `GetData()` = datos

### Flujo Completo Ejemplo
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

---

## Respuesta 6: Interfaz GTK+ (archivos .ui, componentes)

### 1. Archivos .ui y Glade
- **Generación**: Creados con **Glade v3.8** (incluido en TPuy)
- **Manipulación**: Usando librería **T-GTK** para **GTKBuilder**
- **Relación**: Los .ui definen la estructura, XBScript la manipula
- **Ejemplo**: `SET RESOURCES oRes FROM FILE oTPuy:cResources+"vta_facturas.ui"`

### 2. Toolkit Específico
- **Framework**: **T-GTK** (FiveTechSoft) - https://github.com/FiveTechSoft/T-Gtk
- **Base**: GTK+ estándar
- **Integración**: T-GTK proporciona bindings Harbour/GTK

### 3. Manejo de Eventos y Callbacks
- **Sistema**: Basado en **señales GTK** (estándar GTK)
- **Gestión**: Controlado por librerías **T-GTK (gclass)**
- **Fuentes gclass**: https://github.com/FiveTechSoft/T-Gtk/tree/master/src/gclass
- **Ejemplo**: `ACTION __VtaSave( oForm, oFormParent )` en botones
- **Función**: gclass proporciona wrapper de Harbour para clases GTK y gestión de señales

---

## Respuesta 7: Componentes Específicos

### 1. `DEFINE LISTBOX` y `DEFINE MODEL`
- **MODEL**: Define estructura de datos basada en `Struct()`
- **LISTBOX**: Componente visual que muestra los datos del MODEL
- **Relación**: LISTBOX se asocia a MODEL para mostrar datos tabulares

### 2. Métodos de LISTBOX
- **`SetColVisible()`**: Oculta/muestra columnas específicas
- **`SetColTitle()`**: Cambia títulos de columnas
- **`bEdit`**: Callback para edición de filas
- **`bNew`**: Callback para nuevas filas
- **`SetCol()`**: Asigna valores a columnas específicas

### 3. Selección y Edición
- **`bEdit` callback**: Se ejecuta al hacer doble clic o seleccionar
- **Parámetros**: Recibe el modelo (`oModel`) con datos de la fila
- **Edición inline**: Permite modificar directamente en el grid
- **Navegación**: Manejo estándar de GTK para teclado y mouse

---

## Respuesta 8: Programación xHarbour (sintaxis, funciones)

### 1. Funciones de Validación Harbour
- **`hb_isObject(oX)`**: ¿Es oX un objeto? (devuelve .T./.F.)
- **`hb_isNIL(x)`**: ¿Es x nulo? NIL = equivalente a null en otros lenguajes
- **Son funciones estándar Harbour** para validación de tipos

### 2. Manejo de Arrays
- **Declaración**: `aDetal := {}` crea array vacío
- **Sintaxis**: Arrays heterogéneos permitidos: `{"1", 2, .t.}`
- **Tipos**: Pueden contener strings, números, booleanos, objetos
- **Uso común**: Para almacenar detalles de facturas, listas de datos

### 3. PROCEDURE vs FUNCTION
- **PROCEDURE**: No retorna valor (void)
- **FUNCTION**: Sí retorna valor
- **Diferencia clave**: `FUNCTION` puede usar `RETURN valor`, `PROCEDURE` solo `RETURN`

---

## Respuesta 9: Variables y Alcance

### 1. `SET PUBLIC oForm`
- **Significado**: Forma abreviada para instanciar objeto TPublic
- **Equivalencia**: `oForm := TPublic():New()`
- **Propósito**: Crea contenedor dinámico para variables locales del formulario

### 2. Alcance de Variables `::`
- **Viven lo que vive el objeto que las contiene**
- **Ejemplo**: `::cVariable` vive mientras `oForm` exista
- **Alcance**: Limitado al ciclo de vida del objeto TPublic contenedor
- **Acceso**: Solo dentro del contexto del objeto donde se definen

### 3. Variables Globales vs Locales
- **Única global**: `oTPuy` (objeto TPublic global accesible desde cualquier script)
- **Variables locales**: Se declaran al inicio de función/procedimiento
- **Sintaxis**: `LOCAL cNombre, nEdad, aDatos`
- **Alcance**: Solo dentro de la función donde se declaran

---

## Respuesta 10: Flujo del Sistema

### 1. Flujo Exacto desde `begin.xbs`
- **Inicio**: `begin.xbs` → inicializa TPuy
- **Configuración**: tema, interfaz, temporizadores
- **Verificación**: `__NetIOUpdate()` → timer de conexión
- **Ejecución**: `oTPuy:RunXBS("netio_check")` → conecta al servidor
- **Login**: Si conexión OK → `__Login()` → ventana de autenticación
- **Menú principal**: Post-login → sistema funcional

### 2. Gestión de Sesión de Usuario
- **Autenticación**: A través de `netio_login.xbs`
- **Objeto usuario**: `oTPuy:oUser` almacena sesión activa
- **Validación**: Verificación constante de sesión activa
- **Timeout**: Sistema verifica conexión periódicamente

### 3. Manejo de Fallas NetIO
- **Detección**: `netio_check.xbs` verifica conectividad
- **Reintento**: Intenta reconexión automática cada 30 segundos
- **Estados**: `oTPuy:lNetIO` indica estado de conexión
- **Graceful degradation**: Sistema puede funcionar limitadamente sin conexión

---

## Respuesta 11: Desarrollo y Mantenimiento (depuración, testing)

### 1. Depuración de Código XBScript
- **Método tradicional**: A la manera clásica de xHarbour
- **Herramienta principal**: Función `View()` para inspeccionar cualquier variable
- **Uso de View()**: `View( oForm )`, `View( aDatos )`, `View( ::cVariable )`
- **Sin herramientas especiales**: TPuy no incluye depurador específico

### 2. Logs y Traces
- **Logs tradicionales**: Sin sistema específico incluido en TPuy
- **Archivos de log**: Proyecto usa archivos estándar (`comp.log`, `error_.log`)
- **Depuración en tiempo real**: Principalmente con `View()` durante desarrollo

### 3. Herramientas de Desarrollo
- **Editor**: gvim (preferencia personal del desarrollador)
- **Sin IDE específico**: TPuy no requiere herramientas especiales
- **Enfoque minimalista**: Editor de texto + compilación + ejecución
- **Herramientas adicionales**: Si alguien tiene herramientas extra, puede usarlas

---

## Respuesta 12: Convenciones y Estándares

### 1. Prefijos de Archivos
- **`vta_`**: Módulo de Ventas (facturas, clientes, etc)
- **`mae_`**: Maestros/Tablas principales (clientes, inventario, usuarios)
- **`fin_`**: Finanzas (recibos, cajas, pagos)
- **`netio_`**: Conexión y comunicación red
- **Propósito**: Organización modular del sistema

### 2. Prefijo `__` en Funciones
- **Significado**: Funciones internas/privadas del módulo
- **Ejemplo**: `__VtaSave()`, `__fr_HacerModeloEditable()`
- **Uso**: No deberían llamarse directamente desde otros módulos
- **Convención**: Doble guión bajo para funciones auxiliares

### 3. Estándares de Nombres
- **Variables locales**: `cNombre` (c=char, n=numeric, a=array, o=object, l=logical)
- **Variables objeto**: `oForm`, `oBtnGuardar`, `oLBoxClientes`
- **Procedures/Functions**: CamelCase con prefijo de módulo
- **Métodos**: Lowercase para métodos GTK, mixed para funciones Harbour

---

## Respuesta 13: Integración (SENIAT, APIs externas, tasas de cambio)

### 1. Integración SENIAT
- **Estado actual**: De momento no hay nada de parte del SENIAT para integrar
- **Sistema preparado**: Arquitectura permite futuras integraciones
- **Pendiente**: Esperando APIs o sistemas oficiales del SENIAT

### 2. APIs para Impresión Fiscal
- **Existencia**: Hay algo externo disponible
- **Integración**: De momento no hay necesidad de integrarlo
- **Capacidad**: Sistema preparado para incorporar cuando sea necesario

### 3. Tasas de Cambio BCV
- **Manejo**: Lo hace el servidor (fiscalius-server)
- **Cliente**: No gestiona directamente tasas de cambio
- **Arquitectura**: Servidor obtiene datos, cliente los consume vía API

---

## Respuesta 14: Reportes

### 1. Sistema de Reportes
- **Generación**: Uso de librerías específicas para cada formato
- **Excel**: `hbxlsxwriter.ch` para generación de archivos XLSX
- **Impresión**: Sistema tradicional de Harbour + GTK
- **PDF**: No especificado en las respuestas

### 2. `hbxlsxwriter.ch` y Librerías
- **Función**: Include para generar archivos Excel XLSX
- **Uso en proyecto**: `vta_libro_xlsx.xbs`, `vta_imprimir.xbs`
- **Librerías Harbour**: Sistema modular de includes para diferentes formatos

### 3. Diseñador de Reportes
- **Estado**: No hay diseñador visual específico
- **Enfoque**: Generación programática
- **Flexibilidad**: Total control sobre formato y contenido vía código

---

## Resumen Final de Arquitectura

Con estas respuestas, hemos documentado completamente la arquitectura de Fiscalius Client:

- **Framework**: TPuy (Harbour + T-GTK) - desarrollo propio
- **Arquitectura**: 3-capas (Cliente UI → Servidor API → PostgreSQL)
- **Comunicación**: NetIO con transformación transparente remoto→local
- **UI**: Glade + T-GTK con gestión de señales GTK
- **Datos**: Queries remotos → tcursor local → UI
- **Desarrollo**: Minimalista con gvim + View()
- **Integraciones**: Preparado para SENIAT cuando esté disponible

---

## 📋 **INDICE COMPLETO DE RESPUESTAS**

✅ **1. TPuy Framework** - Versión, API, diferencias .ch  
✅ **2. Estructura de Objetos** - Sintaxis ::, jerarquía, instancias  
✅ **3. Conexión NetIO** - Protocolo, mensajes, transformación remota-local  
✅ **4. Base de Datos** - Motores, arquitectura, transacciones  
✅ **5. Modelos de Datos** - Struct(), GetData(), queries remotos  
✅ **6. Interfaz GTK+** - Glade, T-GTK, gclass, señales  
✅ **7. Componentes UI** - DEFINE LISTBOX, MODEL, callbacks  
✅ **8. Programación xHarbour** - hb_isObject(), arrays, FUNCTION vs PROCEDURE  
✅ **9. Variables y Alcance** - SET PUBLIC, ::variables, LOCAL  
✅ **10. Flujo del Sistema** - begin.xbs, NetIO, login, menú  
✅ **11. Desarrollo y Mantenimiento** - Depuración con View(), gvim, logs  
✅ **12. Convenciones** - Prefijos (vta_, mae_), __functions, nomenclatura  
✅ **13. Integraciones** - SENIAT (pendiente), impresión fiscal (externo), BCV (servidor)  
✅ **14. Reportes** - Excel con hbxlsxwriter.ch, sistema programático  

**TOTAL: 14 categorías completadas con respuestas verificadas**

---

## Creación Manual Técnico para Programadores

Con toda esta información, puedo crear un manual técnico completo para desarrolladores nuevos en el proyecto.