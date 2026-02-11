# Guía Rápida de Fiscalius Client

## Resumen del Proyecto
Fiscalius Client es un sistema de facturación electrónica desarrollado en **TPuy** (xHarbour/GT) diseñado para cumplir con las normativas fiscales del SENIAT en Venezuela.

## Estructura del Proyecto

```
fiscalius-client/
├── xbscripts/          # Scripts XBScript (lógica del negocio)
├── resources/          # Interfaces GTK (.ui)
├── images/            # Recursos gráficos
├── include/           # Librerías y headers
├── docs/              # Documentación (esta carpeta)
├── init.conf          # Configuración inicial
├── connect.ch         # Configuración de conexión
└── begin.xbs          # Punto de entrada principal
```

## Tecnologías Clave
- **TPuy**: Framework principal basado en xHarbour
- **GT**: Interfaz gráfica GTK+
- **XBScript**: Lenguaje de scripting del sistema
- **NetIO**: Sistema de conectividad y actualizaciones

## Módulos Principales

### Ventas y Facturación
- `vta_facturas.xbs` - Gestión de facturas
- `vta_clientes.xbs` - Gestión de clientes
- `vta_ncredito.xbs` - Notas de crédito
- `vta_debito.xbs` - Notas de débito

### Maestros
- `mae_clientes2.xbs` - Catálogo de clientes
- `mae_inventario.xbs` - Gestión de inventario
- `mae_monedas.xbs` - Configuración de monedas
- `mae_usuarios.xbs` - Gestión de usuarios

### Finanzas
- `fin_recibo.xbs` - Gestión de recibos
- `fin_cajas.xbs` - Control de cajas

### Sistema
- `netio_check.xbs` - Verificación de conectividad
- `netio_login.xbs` - Autenticación
- `menu.xbs` - Sistema de menús

## Puntos de Entrada
1. **`begin.xbs`**: Inicialización del sistema
2. **`netio_check.xbs`**: Verificación de red y carga inicial
3. **`menu.xbs`**: Interfaz principal de usuario

## Configuración
- **`init.conf`**: Rutas y parámetros básicos
- **`connect.ch`**: Configuración de conexión a bases de datos

## Flujo de Inicio
1. `begin.xbs` inicializa el sistema
2. Configura tema e interfaz
3. Inicia timer de NetIO
4. Ejecuta `netio_check.xbs`
5. Presenta menú principal

## Convenciones
- Archivos `.xbs`: Scripts XBScript
- Archivos `.ui`: Interfaces GTK
- Prefijos: `vta_` (ventas), `mae_` (maestros), `fin_` (finanzas), `netio_` (red)

## Notas Importantes
- Sistema compatible con Windows y Linux
- Requiere TPuy instalado
- Utiliza temporizadores para actualizaciones automáticas
- Soporta múltiples temas visuales

## Primeros Pasos
1. Verificar instalación de TPuy
2. Revisar configuración en `init.conf`
3. Ejecutar con comando `tpuy`
4. Autenticar vía `netio_login.xbs`

---
*Generado para referencia rápida de desarrolladores*