# mae_medios_pago.xbs - Formulario de Medios de Pago

## Descripción
Formulario para gestionar Medios de Pago en Fiscalius.

**Estado**: ✅ Implementado (18/03/2026)

---

## Estructura del Formulario

### Lista Principal
- Muestra todos los medios de pago registrados
- Columnas: Código, Nombre, Comportamiento, IGTF, Afec. C/B, Activo

### Formulario de Edición
Permite crear y modificar medios de pago.

---

## Campos del Formulario

### Campo: Código
```xbscript
DEFINE ENTRY oUsuForm:oEntCodigo VAR oUsuForm:cCodigo ;
       VALID iif( oUsuForm:lClose, .T., __ValCodMP( oUsuForm:oEntCodigo, oUsuForm ) ) ;
       ID "ent_codigo" RESOURCE oRes
```
- Validación: 2-4 caracteres, único en el sistema

### Campo: Nombre
```xbscript
DEFINE ENTRY oUsuForm:oEntNombre VAR oUsuForm:cNombre ID "ent_nombre" RESOURCE oRes
```
- Obligatorio

### Campo: Comportamiento (Actualizado)
Ahora se usa un Combobox para seleccionar el comportamiento:

```xbscript
DEFINE BOX oUsuForm:oBoxComporta ID "hbox_comporta" RESOURCE oRes
DEFINE COMBOBOX oUsuForm:oCBComporta VAR oUsuForm:cComporta ITEMS aComporta;
OF oUsuForm:oBoxComporta  
```

**Opciones disponibles:**
- "Ambos"
- "Suma"
- "Resta"

### Campo: Genera IGTF
```xbscript
DEFINE CHECKBOX oUsuForm:oChkGenITF VAR oUsuForm:lGenITF ID "chk_genitf" RESOURCE oRes
```

### Campo: Afecta Caja/Banco (Actualizado)
Ahora se usa un Combobox para seleccionar:

```xbscript
DEFINE BOX oUsuForm:oBoxAfeCajBco ID "hbox_afecajbco" RESOURCE oRes
DEFINE COMBOBOX oUsuForm:oCBAfeCajBco VAR oUsuForm:cAfeCajBco ITEMS aCajBco ;
OF oUsuForm:oBoxAfeCajBco  
```

**Opciones disponibles:**
- "C" - Caja
- "B" - Banco

### Campo: Activo
```xbscript
DEFINE CHECKBOX oUsuForm:oChkActivo VAR oUsuForm:lActivo ID "chk_activo" RESOURCE oRes
```

---

## Estructura de Datos

### Variables del formulario
```xbscript
oUsuForm:cCodigo       // Código del medio de pago
oUsuForm:cNombre       // Nombre descriptivo
oUsuForm:cComporta     // Comportamiento (Ambos/Suma/Resta)
oUsuForm:lGenITF       // Genera IGTF (Lógico)
oUsuForm:cAfeCajBco    // Afecta Caja o Banco (C/B)
oUsuForm:lActivo       // Activo (Lógico)
```

### Array de opciones
```xbscript
aCajBco := {"C", "B"}
aComporta := {"Ambos", "Suma", "Resta"}
```

---

## Guardar Datos
```xbscript
oForm:rMedioPago:SetData( { "codigo"        => oForm:cCodigo,   ;
                             "nombre"       => oForm:cNombre,  ;
                             "comportamiento" => oForm:cComporta, ;
                             "genitf"       => oForm:lGenITF,  ;                             
                             "activo"       => oForm:lActivo,  ;
                             "afecajbco"    => oForm:cAfeCajBco } )
```

---

## Base de datos
```sql
-- Tabla: mae_medios_pago
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | integer | ID único |
| codigo | varchar(4) | Código único |
| nombre | varchar(50) | Nombre descriptivo |
| comportamiento | varchar(10) | Ambos/Suma/Resta |
| genitf | boolean | Genera IGTF |
| activo | boolean | Está activo |
| afecajbco | char(1) | C=Caja, B=Banco |
```
