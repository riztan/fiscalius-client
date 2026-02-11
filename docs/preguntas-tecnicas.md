# Preguntas Técnicas - Fiscalius Client

## Arquitectura y Framework

### 1. TPuy Framework
- ¿Qué versión específica de TPuy se está utilizando?
- ¿Dónde está documentada la API de TPuy (clases, métodos)?
- ¿Cuál es la diferencia entre `tpy_xbs.ch` y `tpy_class.ch`?

### 2. Estructura de Objetos
- ¿Qué representa `::` (doble dos puntos) en el código? ¿Es self/this?
- ¿Cuál es la jerarquía de clases principal? (`oGEmpresa`, `rVentas`, `rFacturas`)
- ¿Cómo se crea una nueva instancia de objeto TPuy?

### 3. Conexión NetIO
- ¿Qué protocolo usa NetIO? ¿TCP/IP personalizado?
- ¿Dónde se define la estructura de mensajes entre cliente y servidor?
- ¿Cómo se maneja la autenticación y seguridad en NetIO?

## Base de Datos

### 4. Estructura de Datos
- ¿Qué motor de base de datos se utiliza (MySQL, PostgreSQL, etc)?
- ¿Dónde están las definiciones de tablas y esquemas?
- ¿Cómo se realizan las transacciones y manejo de concurrencia?

### 5. Modelos de Datos
- ¿Qué hace `rClientes:Struct()`? ¿Retorna metadata de la tabla?
- ¿Cómo funciona `rVentas:Clientes()`? ¿Es un ORM?
- ¿Cuál es la estructura de los datos retornados por `GetData()`?

## Interfaz Gráfica

### 6. GTK+ y Resources
- ¿Cómo se relacionan los archivos `.ui` con el código XBScript?
- ¿Qué toolkit específico de GTK se está utilizando?
- ¿Cómo se manejan eventos y callbacks en la UI?

### 7. Componentes Específicos
- ¿Cómo funciona `DEFINE LISTBOX` y `DEFINE MODEL`?
- ¿Qué métodos tiene un objeto LISTBOX (SetColVisible, SetColTitle, etc)?
- ¿Cómo se maneja la selección y edición en grid/listas?

## Programación Específica

### 8. Sintaxis xHarbour
- ¿Qué significa `hb_isObject()`, `hb_isNIL()`? ¿Son funciones estándar Harbour?
- ¿Cómo funciona el manejo de arrays con `aDetal := {}`?
- ¿Cuál es la diferencia entre `PROCEDURE` y `FUNCTION` en este contexto?

### 9. Variables y Alcance
- ¿Qué significa `SET PUBLIC oForm`?
- ¿Cuál es el alcance de las variables `::`?
- ¿Cómo se manejan las variables globales vs locales?

## Flujo del Sistema

### 10. Ciclo de Vida
- ¿Cuál es el flujo exacto de inicio desde `begin.xbs`?
- ¿Cómo se gestiona la sesión de usuario?
- ¿Qué pasa cuando falla NetIO?

### 11. Módulos
- ¿Cómo se registra un nuevo módulo en el sistema?
- ¿Dónde se definen los permisos por módulo?
- ¿Cómo funciona el sistema de menús dinámicos?

## Desarrollo y Mantenimiento

### 12. Depuración
- ¿Cómo se depura el código XBScript?
- ¿Hay logs o traces disponibles?
- ¿Qué herramientas se usan para desarrollo?

### 13. Testing
- ¿Hay framework de testing?
- ¿Cómo se prueban los componentes UI?
- ¿Hay ambiente de desarrollo vs producción?

## Convenciones y Estándares

### 14. Nomenclatura
- ¿Por qué algunos prefijos son `vta_`, `mae_`, `fin_`?
- ¿Qué significa el prefijo `__` en funciones como `__VtaSave()`?
- ¿Hay estándares de nombres para variables y métodos?

### 15. Patrones de Diseño
- ¿Se sigue algún patrón MVC o similar?
- ¿Cómo se maneja la validación de datos?
- ¿Qué patrón se usa para manejo de errores?

## Integración

### 16. APIs Externas
- ¿Cómo se integra con sistemas fiscales del SENIAT?
- ¿Hay APIs para impresión fiscal?
- ¿Cómo se manejan las tasas de cambio BCV?

### 17. Reportes
- ¿Qué sistema se usa para generar reportes (imprimir, PDF, Excel)?
- ¿Cómo funcionan `hbxlsxwriter.ch` y otros includes de reportes?
- ¿Hay diseñador de reportes visual?

---
*Por favor responder en el orden que prefieran. Puedo trabajar con respuestas parciales.*