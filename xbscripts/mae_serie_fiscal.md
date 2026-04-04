# mae_serie_fiscal - Series Fiscales

## Descripción
Formulario para gestionar las series fiscales en Fiscalius.

**Estado**: ✅ Implementado (02/04/2026)

## Tabla
- **Schema**: `orseit`
- **Tabla**: `mae_serie_fiscal`

---

## Componentes

### Servidor
- **Clase**: `SerieFiscal` (`tseriefiscal.prg`)
- **Métodos**:
  - `New()` - Constructor
  - `SetValue()` - Asignar valores
  - `Save()` - Guardar serie
  - `GetNextSecuencia()` - Obtener siguiente número de secuencia
- **En empresa.prg**:
  - `SerieFiscal()` - Lista todas las series
  - `GetSerieFiscal(hParam)` - Obtener serie por ID

### Cliente
- **Formulario**: `mae_serie_fiscal.xbs`
- **UI**: `mae_serie_fiscal.ui`

---

## Campos del Formulario

| Campo | Descripción |
|-------|-------------|
| Empresa | ID de empresa |
| Sucursal | ID de sucursal |
| Serie | Serie fiscal (A, B, etc.) |
| Inicio | Secuencia inicial |
| Fin | Secuencia final |
| Activa | Si está activa |

---

## Columnas en ListBox

| # | Campo | Descripción |
|---|-------|-------------|
| 1 | ID | ID único (oculto) |
| 2 | Empresa | ID empresa |
| 3 | Sucursal | ID sucursal |
| 4 | Serie | Serie fiscal |
| 5 | Ini | Secuencia inicio |
| 6 | Actual | Secuencia actual |
| 7 | Fin | Secuencia fin |
| 8 | Activa | Si está activa |

---

## Tablas Relacionadas

- `orseit.mae_serie_fiscal` - Maestro de series
- `orseit.mae_serie_tipos` - Tipos de documento por serie

---

## Archivos Creados

### Servidor
- `apps/fiscalius/tseriefiscal.prg` - Clase SerieFiscal
- `apps/fiscalius/empresa.prg` - Métodos SerieFiscal() y GetSerieFiscal()

### Cliente
- `xbscripts/mae_serie_fiscal.xbs` - Formulario
- `resources/mae_serie_fiscal.ui` - UI

---

## Notas
- La funcionalidad de eliminar serie está comentada (en desarrollo)
- Falta implementar la gestión de `mae_serie_tipos` (tipos de documento)
