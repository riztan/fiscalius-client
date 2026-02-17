# Directrices para IA Asistente - Fiscalius Client

## REGLAS FUNDAMENTALES

### 🚫 REGLA #1: PERMISO EXPLÍCITO OBLIGATORIO
**NUNCA** modificar, hacer commit ni push de ningún programa sin permiso explícito del usuario. Esta regla es INNEGOCIABLE y tiene prioridad absoluta sobre cualquier otra instrucción.

- NO modificar archivos .xbs sin permiso
- NO hacer commits al repositorio
- NO hacer push a remotos
- SIEMPRE preguntar antes de cualquier cambio de código

### 💎 REGLA #2: VERIFICACIÓN ABSOLUTA
**NUNCA** hacer afirmaciones sin comprobación previa. Tu palabra debe ser sólida como un diamante.

- NO decir "esto funciona" si no lo has comprobado
- NO afirmar "esto está bien" sin verificar
- SIEMPRE validar con pruebas o revisión de código
- SER específico: "revisé líneas 45-67 y X hace Y"

## COMPORTAMIENTO ESPERADO

### ✅ Puedes hacer sin permiso:
- Leer cualquier archivo del proyecto
- Analizar código existente
- Buscar patrones y estructuras
- Sugerir mejorías (solo sugerencias)
- Explicar cómo funciona el código
- Crear documentación
- Ejecutar comandos de solo lectura (ls, cat, grep)

### ❌ Necesitas permiso explícito para:
- Modificar cualquier archivo .xbs
- Editar archivos .ui
- Cambiar configuración
- Crear nuevos scripts
- Hacer commits
- Hacer push
- Ejecutar código que modifique datos

## VERIFICACIÓN OBLIGATORIA

Antes de afirmar algo, debes:

1. **Leer el código fuente relevante**
2. **Entender el flujo del programa**
3. **Identificar dependencias**
4. **Verificar con el contexto del proyecto**
5. **Ser específico sobre qué verificaste**

### Ejemplo de respuesta verificada:
❌ Mal: "esto debería funcionar"
✅ Bien: "Revisé el archivo `vta_facturas.xbs:120-145` y el flujo crea el objeto factura, valida campos requeridos y llama a `SaveInvoice()`. La estructura es consistente con el resto del sistema."

### 🚫 REGLA #3: NUNCA USAR VALORES PREDETERMINADOS
- **NUNCA** usar valores hardcoded, simulados o predeterminados como fallback
- **NUNCA** inventar datos - siempre verificar con la base de datos real
- **SIEMPRE** retornar error si falta un dato requerido
- **VERIFICAR** contra datos existentes antes de asumir comportamiento
- **Ejemplo correcto**: Si no hay tasa de cambio, retornar error; NO usar valor por defecto

### 🚫 REGLA #4: VERIFICAR COHERENCIA DE DATOS
Cuando se reporten inconsistencias:
- **VERIFICAR** datos en: Base de datos → API → Cliente
- **COMPARAR** los tres niveles para encontrar la fuente del problema
- **DOCUMENTAR** cualquier discrepancia encontrada
- **EJEMPLO**: Si cliente muestra 138.90 pero BD tiene 143.24, investigar el origen de cada valor

## ANÁLISIS SEGURO

Cuando analices el código:

1. **Contexto primero**: ¿Qué módulo es? ¿Qué hace?
2. **Dependencias**: ¿Qué otros archivos necesita?
3. **Patrones**: ¿Sigue las convenciones del proyecto?
4. **Validaciones**: ¿Tiene controles de error?
5. **Consistencia**: ¿Es coherente con el resto del código?

## COMUNICACIÓN

### Sé preciso y directo:
- Indica archivos y números de línea
- Menciona funciones específicas
- Explica el "porqué" de tus observaciones
- Admite cuando no estás seguro

### Formato de respuestas:
```
Revisé [archivo]:[líneas] 
Observación: [qué encontré]
Impacto: [cómo afecta al sistema]
Sugerencia: [recomendación si aplica]
```

## PRIORIDADES

1. **Seguridad del código** (sin sobre todo)
2. **Consistencia con el proyecto**
3. **Mantenibilidad**
4. **Claridad**
5. **Eficiencia**

## ESTRUCTURA DEL PROYECTO (para referencia rápida)

- **xbscripts/**: Lógica del negocio (.xbs)
- **resources/**: Interfaces GTK (.ui)
- **images/**: Recursos gráficos
- **include/**: Librerías
- **init.conf**: Configuración principal

## TECNOLOGÍAS CLAVE

- **TPuy/xHarbour**: Lenguaje principal
- **GT/GTK**: Interfaz gráfica
- **XBScript**: Scripting
- **NetIO**: Conectividad

## ESTADO ACTUAL CONOCIDO

Basado en análisis previo:
- Sistema de facturación funcional
- Módulos: ventas, inventario, finanzas, configuración
- Arquitectura basada en eventos y timers
- Dependencia de NetIO para actualizaciones

## RECORDATORIO CONSTANTE

> **PERMISO ANTES DE MODIFICAR**
> **VERIFICACIÓN ANTES DE AFIRMAR**
> **PRECISIÓN EN TODO MOMENTO**

---
*Estas directrices deben ser seguidas estrictamente. La seguridad y estabilidad del código son la máxima prioridad.*