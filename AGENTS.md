# AGENTS.md — Fiscalius Client

Cliente TPuy/xHarbour + GTK para el ERP fiscal venezolano Fiscalius.
**No accede a la BD directamente** — toda la lógica está en el servidor vía NetIO (hb_netio TCP/IP, puerto 2942).

## Comandos

```bash
tpuy                            # Ejecutar desde raíz del proyecto
```

## Reglas

- NO modificar `.xbs` o `.ui` sin permiso explícito
- NO hacer commits/push sin autorización
- Si detectas un problema en Y mientras trabajas en X, informa pero NO modifiques Y sin autorización
- Si en ~1 minuto no encuentras lo que buscas, pregúntale al usuario
- NUNCA inventar funciones o procedimientos — verifica que existen con grep en `xbscripts/`

## Harbour — Errores silenciosos

| Regla | ❌ Incorrecto | ✅ Correcto |
|-------|--------------|-------------|
| `LOCAL` al inicio | `FOR EACH ...; local nLen := ...; NEXT` | `local aLine, nLen, aVal` al inicio del procedure |
| `::` en procedures | Usar `::cVar` en function/procedure | `SET PUBLIC oForm` → `oForm:cVar` |
| Conversión numérica | `VAL(cValor)` pierde decimales | `ToNum(cValor)` siempre |
| `APPEND LIST_STORE` valores | `VALUES aLine[1], aLine[1]/aLine[5]` | `VALUES aLine[1], aLine[2]` (solo strings, calcular antes) |
| Acceso a objetos/hash | `oObj:campo` sin verificar | `hb_isObject(oObj) .and. !hb_isNIL(oObj:campo)` |
| `hb_hGetDef` | No existe en Harbour de TPuy | `hb_HHasKey(h, "k")` + acceso directo |
| `ENDMETHOD` | No existe en xHarbour | `RETURN` (sin `ENDMETHOD`) |
| Deshabilitar botón UI | `::oBtn:Disable()` antes de `DEFINE BUTTON` | Después de la definición del botón |

## NetIO — Patrón de llamada al servidor

```xbase
// 1. Verificar conexión
if !oTPuy:RunXBS("netio_check")
   MsgStop("Sin conexión al servidor"); RETURN
endif

// 2. Obtener módulo remoto
::rVentas := oFormParent:oGEmpresa:SetModulo("Ventas")

// 3. Llamar método (retorna tcursor local transparente)
oClientes := ::rVentas:Clientes( hParams )

// 4. Cargar datos al ListBox
oModel:oTreeView:ClearModel(.T.)
FOR EACH aLine IN oClientes:GetData()
   APPEND LIST_STORE oModel:oLbx ITER aIter VALUES aLine[1], aLine[2], aLine[3]
NEXT
```

Sintaxis alternativa (de `tpy_netio.ch`):
- `~objeto:msg(params)` → `FromRemote("__object", objeto, #msg, params)`
- `r:objeto:msg` → `FromRemote("__objmethod", objeto, #msg)`

## UI GTK (.ui) — Orden crítico

```xbase
SET RESOURCES ::oRes FROM FILE oTPuy:cResources+"vta_facturas.ui"
DEFINE WINDOW ::oFacWnd TITLE "Fiscalius. Factura" SIZE 800,600 ID "window1" RESOURCE ::oRes
DEFINE BUTTON ::oBtnGuardar ID "btn_guardar" RESOURCE ::oRes
```

- `SetColVisible`, `SetColTitle` debe ir **antes** de `Active()`
- Deshabilitar botones (Disable) **después** de `DEFINE BUTTON`
- Permisos: Admin/Devel acceso total; Propio usuario: solo cambio contraseña; Auditor/Demo: sin acceso

## Flujo de entrada

`xbscripts/begin.xbs` → timer `__NetIOUpdate()` cada 1s → `netio_check.xbs` → login → `poslogin.xbs` (session, empresa, fuentes) → `menu.xbs`

Archivo `devel` (vacío) en raíz → salta sincronización de archivos en login (modo desarrollo).

## Archivos de configuración clave

| Archivo | Contenido |
|---------|-----------|
| `connect.ch` | `NETSERVER`, `NETPORT` (2942), `NETPASSWD` — incluido por `netio_check.xbs` vía `#include` en tiempo de ejecución |
| `init.conf` | `oTPuy:cResources`, `cImages`, `cXBScript`, `nDecimals := 2` |

## Prefijos de procedimientos

`vta_` (ventas), `mae_` (maestros), `fin_` (finanzas), `netio_` (conexión), `__` (interno del módulo)

## Referencias útiles (no reescribir aquí)

| Archivo | Contenido |
|---------|-----------|
| `.opencode/skills/fiscalius-client/SKILL.md` | Skill completo con objetos remotos y patrones |
| `.github/copilot-instructions.md` | Procedimientos verificados y objetos remotos |
| `.cursor/rules/fiscalius.md` | Reglas detalladas para Cursor IDE |
| `docs/directrices-ia.md` | Reglas de comportamiento IA |
| `docs/SESION_MAYO_2026.md` | Problemas conocidos (BASE/1132, hb_hGetDef, permisos) |
