# vta_facturas.xbs - Condición de Pago

## Descripción
Se agregó el campo de condición de pago (contado/crédito) en el formulario de facturas.

**Estado**: ✅ Implementado (02/04/2026)

---

## Campo: condición_id

### Definición en UI (vta_facturas.ui)
```xml
<object class="GtkBox" id="box_condicion">
  <child>
    <object class="GtkComboBox" id="combo_condicion">
    </object>
  </child>
</object>
```

### Definición en Código (vta_facturas.xbs)

#### Array de opciones
```xbscript
::aCondiciones := {"Contado", "Crédito"}
::cCondicion := "Contado"
```

#### ComboBox
```xbscript
DEFINE COMBOBOX ::oComboCondicion ;
       VAR ::cCondicion ITEMS ::aCondiciones ;
       ON CHANGE MsgInfo("condicion") ;
       OF ::oBoxCondicion
```

#### Asignación al guardar
```xbscript
::hFactura["condicion_id"     ] := iif(::cCondicion="Contado","1","2")
```

---

## Valores

| Valor | Descripción |
|-------|-------------|
| 1 | Contado |
| 2 | Crédito |

---

## Tabla en Base de Datos
- **Tabla**: `orseit.vta_transacciones`
- **Campo**: `condicion_id` (character varying)

---

## Archivos Modificados
- `xbscripts/vta_facturas.xbs` - Agregado combo y asignación
- `resources/vta_facturas.ui` - UI del combo

---

## Notas
- El valor se guarda como "1" (Contado) o "2" (Crédito)
- En la impresión de factura se muestra la condición usando:
  ```xbscript
  IF ::hFactura["condicion_id"] > 1
     // Es crédito
  ENDIF
  ```
