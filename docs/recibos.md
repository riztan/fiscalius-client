# Programa de Recibos de Cobro - Documentación de Desarrollo

## Estado: En Desarrollo

---

## 1. Archivos Involucrados

### Cliente (`xbscripts/`)
- `fin_recibo.xbs` - Formulario principal de recibos de cobranza
- `fin_cajas.xbs` - Formulario CRUD de cajas (gestion de cajas)
- `fin_cajas.ui` - Interfaz GTK para formulario de cajas

### Servidor (`apps/fiscalius/`)
- `tf_recibo.prg` - Clase TFRECIBO para gestion de recibos
- `tf_caja.prg` - Clase TFCAJA para gestion de cajas
- `finanzas.prg` - Metodos Cobranza() y Caja()

---

## 2. Flujo de Trabajo

### 2.1 Apertura del Formulario
1. Usuario abre `fin_recibo.xbs`
2. Se crea objeto `::rCobranza := ::rFinanzas:Cobranza('')` al inicio
3. Se carga lista de documentos pendientes del cliente via `CxCxFlujo()`

### 2.2 Seleccion de Documentos
- Usuario selecciona facturas en `::oModDocuments` (TreeView)
- Columna 16 = seleccionado
- Columna 6 = transaccion_id
- Columna 13 = monto_aplicado

### 2.3 Medios de Pago
- Usuario registra medios de pago en `::oModMedios`
- Columna 1 = medio_pago_id (codigo)
- Columna 3 = monto
- Columna 4 = soporte_nro

### 2.4 Guardado
1. Validar saldo = 0
2. Validar equilibrio: monto docs = monto instrumentos
3. Enviar datos via `::rCobranza:SetData(hData)`
4. Guardar via `::rCobranza:Save()`

---

## 3. Gestion de Cajas

### 3.1 Formulario fin_cajas.xbs
- Permite agregar, editar cajas
- Seleccionar caja desde otro formulario (ej. fin_recibo)
- UI: fin_cajas.ui (generado con DeepSeek/Glade)

### 3.2 Estructura de Caja
- codigo (varchar 20) - unico por empresa
- nombre (varchar 100)
- moneda_principal (varchar 3) - VES, USD, etc.
- activa (boolean)

---

## 4. Pendientes

- Testing completo de guardado de recibos
- Manejo de recibos existentes (cargar y modificar)
- Integracion con flujos de documentos (IVA, ISLR)

---

## 5. API - Metodos

### TFRECIBO
- `Data()` - Retorna hash con datos del recibo
- `SetData(hData)` - Actualiza datos
- `Save()` - Guarda en BD
- `SetAplicaciones(aApps)` - Aplica pagos a documentos
- `SetInstrumentos(aInsts)` - Define medios de pago

### TFCAJA
- `Data()` - Retorna hash con datos de la caja
- `SetData(hData)` - Actualiza datos
- `Save()` - Guarda en BD
- `IsNew()` - Indica si es nueva caja
