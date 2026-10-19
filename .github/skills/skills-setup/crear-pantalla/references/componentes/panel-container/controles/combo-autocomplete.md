# Combo / Autocomplete

## Proposito

`CommonAC` (`src/components/CommonAC.jsx`) y `MediCommonAC`
(`src/medi/mediComponents/MediCommonAC.jsx`) envuelven `Autocomplete` de
Material UI Lab. Preferir la variante Medi en pantallas de ese modulo. No
mezclar ambas variantes en la misma pantalla sin justificar.

## Rol

Puede cumplir dos roles a la vez (ver tabla de roles en
[panel-container.md](../panel-container.md#roles-de-un-control-dentro-del-panel)):

- **Selector con datos propios**: sus opciones vienen de un mock/GET
  separado del listado principal (ver [Servicio mock](../../../servicio-mock.md)).
  Ejemplo real: `src/medi/CalidadGas/CopiaVariablesCalidad/hooks/useAreas.js`
  llama a `Service.getAreasCalidad()` para poblar el combo, independiente del
  service que trae los datos de la tabla principal.
- **Parametro de consulta**: el valor elegido se envia al service principal
  al listar.

## Etiqueta compuesta por varios campos

Las opciones no siempre son strings. Si el mock/GET devuelve objetos
(ej. `{ numeroPM: "5", nombre: "GBA-tgs" }`), componer el texto visible con
`getOptionLabel`, prop ya soportada por ambos componentes y reenviada al
`Autocomplete` interno (no hay que agregar nada nuevo para esto):

```jsx
<MediCommonAC
    options={areas}
    getOptionLabel={area => `${area.numeroPM} - ${area.nombre}`}
    value={areaSeleccionada}
    onChange={(event, value) => setAreaSeleccionada(value)}
/>
```

Esto funciona igual sin importar cuantos objetos devuelva el mock (1, 100,
N): `getOptionLabel` se aplica por item, no depende del tamano del array.

## Props minimas verificadas

| Prop | Tipo | Descripcion |
|---|---|---|
| options | Array | Opciones disponibles |
| value | any | Opcion seleccionada |
| onChange | function(event, value, name) | Callback de seleccion |
| getOptionLabel | function(option) | Texto visible por opcion |
| disabled | boolean | Deshabilita el control |
| label | string | Etiqueta del campo |
| loading | boolean | Muestra un spinner en el campo |
| showNavigationButtons | boolean | Agrega flechas para navegar entre opciones (solo `MediCommonAC`/`CommonAC` con esa prop) |

## Cuando NO usarlo

- No hardcodear un solo campo del objeto como label si el requerimiento pide
  combinar mas de uno.
- No asumir que valida duplicados o unicidad de opciones.
- No usarlo para una fecha: ver [selector-fecha.md](./selector-fecha.md).
