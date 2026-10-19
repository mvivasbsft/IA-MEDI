# Boton Listar

## Proposito

`CommonListarButton` (`src/components/CommonListarButton.jsx`) y
`MediCommonListarButton` (`src/medi/mediComponents/MediCommonListarButton.jsx`)
son el boton de accion que dispara la consulta principal del panel.

## Rol

Accion: al hacer click, llama al service principal (mock en este piloto) con
los parametros ya resueltos por los demas controles del panel (ver
[panel-container.md](../panel-container.md#roles-de-un-control-dentro-del-panel)).
No es un filtro ni un selector; no tiene datos propios.

## Distribucion por defecto

Patron verificado en 37 pantallas de
`src/medi/**/components/containers/PanelContainer.jsx`: una fila con dos
zonas, controles a la izquierda y el boton alineado a la derecha. Ejemplo
real:
`src/medi/Reportes/ConsultaAguaAzufreMetanol/components/containers/PanelContainer.jsx`
(`Grid item` ancho 9/12 para los controles, `Grid item` ancho 3/12 con
`style={{ justifyContent: "flex-end" }}` para el boton). Ver la tabla de
distribucion por defecto en
[panel-container.md](../panel-container.md#resolver-la-distribucion-espacial-grid).

## Props minimas verificadas

| Prop | Tipo | Descripcion |
|---|---|---|
| onClick | function | Dispara la consulta |
| disabled | boolean | Deshabilita el boton (ej. mientras carga o falta un campo requerido) |
| text | string | Texto visible, por defecto "LISTAR" |
| noIcon | boolean | Oculta el icono |
| height / width | number | Dimensiones del boton |

## Cuando NO usarlo

- No asumir que siempre hay un unico boton Listar por panel, pero tampoco
  asumir por default que no hace falta: en el 77% de los paneles reales del
  repo existe. Confirmarlo con el dev como parte del roster simple de
  controles (ver [panel-container.md](../panel-container.md#resolver-controles-de-forma-simple-base-de-crear-pantalla)),
  no decidirlo en ningun sentido sin preguntar.
- No confundirlo con las acciones de `CommonActionBar` (Agregar/Cancelar/
  Grabar), que operan sobre el estado de edicion, no sobre la consulta (ver
  [action-bar.md](../../action-bar.md)).
