# Filtro de texto local

## Proposito

`TextField` simple de Material UI que recorta, del lado del cliente, los
registros que ya trajo el servicio. Usarlo solo cuando el dev confirma que el
campo es texto libre; no es un default automatico para cualquier columna de
datos (ver [panel-container.md](../panel-container.md#resolver-controles-de-forma-simple-base-de-crear-pantalla)).

## Rol

Filtro local post-fetch (ver tabla de roles en [panel-container.md](../panel-container.md#roles-de-un-control-dentro-del-panel)).
No tiene origen de datos propio: opera sobre los registros que ya llegaron a
la tabla, no vuelve a pedirle nada al servicio.

## Contrato

Comparacion por inclusion ("contiene"), sin distinguir mayusculas ni
minusculas, sobre el valor de cada campo de datos. No aplica a columnas de
accion (Editar, Borrar, etc.). No agregar un control para limpiar filtros
salvo que el dev lo pida explicitamente: no es un patron verificado del
proyecto (0 de 48 `PanelContainer.jsx` reales lo tienen).

## Ejemplo minimo (verificado)

```jsx
const coincideFiltro = (valor, filtro) =>
    !filtro || String(valor).toLowerCase().includes(filtro.toLowerCase());

<TextField label="PM" value={filters.PM} onChange={handleFiltroChange('PM')} />
```

Patron minoritario en el repo (0 de 48 `PanelContainer.jsx` reales lo usan sin
boton Listar): usarlo solo cuando el dev lo confirma explicitamente como
parte del roster simple de controles, no como default automatico para
"cualquier columna de datos".

## Cuando NO usarlo

- No usarlo para rangos, normalizacion especial o reglas remotas: ahi
  corresponde [selector-fecha.md](./selector-fecha.md) u otra referencia segun
  el caso.
- No filtrar columnas de accion.
- No asumir que existe busqueda interna de `MediCommonTable` (`search`) como
  sustituto: ese buscador recorre valores crudos de celda, no reemplaza un
  filtro por campo (ver [tabla.md](../../tabla.md)).
