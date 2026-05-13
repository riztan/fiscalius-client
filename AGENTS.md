# AGENTS.md - Fiscalius Client

**Proyecto**: Cliente TPuy/xHarbour + GTK para el ERP fiscal venezolano Fiscalius
**Directorio**: `/home/riztan/git/fiscalius-client`

---

## Ejecución

```bash
tpuy  # Desde el directorio del proyecto
```

El cliente no accede a la base de datos directamente. Toda la lógica de negocio está en el servidor.

---

## Reglas Absolutas

- **NUNCA** modificar archivos `.xbs` o `.ui` sin permiso explícito del usuario
- **NUNCA** hacer commits o push sin autorización
- **NUNCA** usar valores hardcoded o inventar datos
- **SIEMPRE** verificar coherencia: BD = API = Cliente

---

## LOCAL en Harbour — Regla Crítica

`LOCAL` debe declararse **AL INICIO** del procedimiento, antes de cualquier código ejecutable.

```harbour
// ✅ Correcto
procedure MiProc()
   local aLine, nLen, aVal

   FOR EACH aLine IN aData
      nLen := Len( aLine )
   NEXT
end

// ❌ Error E0004 - LOCAL dentro del bucle
procedure MiProc()
   FOR EACH aLine IN aData
      local nLen := Len( aLine )
   NEXT
end
```

---

## Conversión Numérica

Usar **siempre `ToNum()`** para campos con decimales. `VAL()` puede perder decimales.

```xbase
nValor := ToNum( cValor )   // ✅ Correcto
nValor := VAL( cValor )     // ❌ Incorrecto
```

---

## LIST_STORE — Solo Strings

El widget `APPEND LIST_STORE` espera **todos los valores como strings**. No hacer operaciones matemáticas dentro de VALUES.

```harbour
// ✅ Correcto - pasar valores ya formateados
APPEND LIST_STORE ::oModDetal:oLbx ITER aIter VALUES aLine[21], aLine[22], aLine[23]

// ❌ Incorrecto - operaciones dentro de VALUES
APPEND LIST_STORE ::oModDetal:oLbx ITER aIter VALUES aLine[21], aLine[21]/aLine[5], aLine[5]
```

---

## Acceso a Objetos — Verificar Antes

Error "condicional: nil" (BASE/1123) cuando se accede a propiedades de objetos nulos:

```xbase
// ✅ Verificar antes
if hb_isObject( oObj ) .and. !hb_isNIL( oObj:campo )
   cValor := oObj:campo
endif

// ✅ Acceso en cursor
if !oQry:Eof() .and. !hb_isNIL( oQry:fecha )
   cValor := oQry:fecha
endif
```

---

## Patrón de Llamada al Servidor (NetIO)

```xbase
// 1. Verificar conexión
if !oTPuy:RunXBS("netio_check")
   MsgStop("Sin conexión al servidor")
   RETURN
endif

// 2. Obtener objeto remoto
::rFinanzas := oFormParent:oGEmpresa:SetModulo("Finanzas")

// 3. Llamar método (respuesta transparente como tcursor)
oClientes := ::rVentas:Clientes( hParams )

// 4. Cargar datos al ListBox
oModel:oTreeView:ClearModel(.T.)
FOR EACH aLine IN oClientes:GetData()
   APPEND LIST_STORE oModel:oLbx ITER aIter VALUES aLine[1], aLine[2], aLine[3]
NEXT
```

---

## Estructura del Proyecto

```
fiscalius-client/
├── xbscripts/          # Lógica del negocio (.xbs)
│   ├── begin.xbs       # Punto de entrada
│   ├── vta_*.xbs       # Módulo Ventas
│   ├── mae_*.xbs       # Maestros (inventario, clientes…)
│   └── fin_*.xbs       # Módulo Finanzas
├── resources/          # Interfaces GTK (.ui)
├── images/             # Recursos gráficos
└── init.conf           # Rutas y configuración
```

---

## Módulos Principales

| Script | Función |
|--------|---------|
| `vta_facturas.xbs` | Facturación |
| `vta_notas_cd.xbs` | Notas crédito/débito |
| `vta_notas_entrega.xbs` | Notas de entrega |
| `fin_recibo.xbs` | Recibos de cobranza |
| `fin_cajas.xbs` | Cajas registradoras |
| `mae_inventario.xbs` | Catálogo productos |

---

## Prefijos de Procedimientos

- `vta_` → Ventas, `mae_` → Maestros, `fin_` → Finanzas
- `__` → Procedimiento interno del módulo (no llamar desde afuera)

---

## Tipos de Documentos Fiscales

- `tipo_id = 4`: Factura (serie A)
- `tipo_id = 5`: Pedido (serie B)
- `tipo_id = 6`: Nota Crédito (serie B)
- `tipo_id = 14`: Nota Entrega (serie C)

---

## Tablas Clave del Servidor

- `vta_serie_fiscal`: maestro de series (A, B, C, '')
- `vta_serie_tipos`: vincula series a tipos de documentos
- `vta_serie_consecutivos`: correlativos de control por serie_id

---

## Variables Globales

- `oTPuy` → Objeto TPublic global, accesible desde cualquier script

---

## Verificaciones Obligatorias

1. Antes de afirmar algo, leer el código fuente y cite archivo:líneas
2. Si no encuentra algo en ~1 minuto, pregunte al usuario
3. Verificar con la BD real antes de asumir comportamiento

---

## Documentación de Referencia

| Archivo | Contenido |
|---------|-----------|
| `docs/directrices-ia.md` | Reglas del proyecto (LEER SIEMPRE) |
| `.opencode/skills/fiscalius-client/SKILL.md` | Skill completo con arquitectura |
| `docs/guia-rapida.md` | Mapa del proyecto |
| `CHANGELOG.md` | Historial de cambios |
| `docs/SESION_MAYO_2026.md` | Resumen de sesión de trabajo (leer para retomar) |

---

## Proyecto Completo (Cliente + Servidor)

Para retomar contexto del servidor, leer en el **servidor**:

- `AGENTS.md` - Reglas del servidor y comandos de compilación
- `SESION_TRABAJO.md` - Historial de sesiones de trabajo
- `docs_ia/QUICK_CONTEXT.md` - Resumen rápido del estado actual
- `docs_ia/TAREAS_PENDIENTES.md` - Roadmap y deudas técnicas
- `docs_ia/PREGUNTAS_ABIERTAS.md` - Preguntas respondidas del proyecto
- `work/registrar-notas-entrega/` - Tareas pendientes para notas de entrega |
