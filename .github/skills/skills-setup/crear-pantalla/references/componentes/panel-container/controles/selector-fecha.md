# Selector de fecha

## Proposito

`CommonDatePicker` (`src/components/CommonDatePicker.jsx`) y su variante
`MediCommonDatePicker` (`src/medi/mediComponents/MediCommonDatePicker.jsx`)
envuelven `KeyboardDatePicker` de Material UI Pickers. Preferir la variante
Medi en pantallas de ese modulo (ya trae estilos propios ajustados). No
mezclar ambas variantes en la misma pantalla sin justificar.

## Rol

Normalmente actua como parametro de consulta (ver tabla de roles en
[panel-container.md](../panel-container.md#roles-de-un-control-dentro-del-panel)):
su valor se envia al service cuando se dispara la accion principal, no
filtra datos ya mostrados. Ejemplo real: `selectedDate` en
`src/medi/Reportes/ConsultaAguaAzufreMetanol/Context.jsx`, usado como
`Service.listar(selectedDate)`.

## Cardinalidad: fecha puntual vs rango

Puede haber 1 instancia (fecha puntual) o 2 instancias (rango Desde/Hasta).
Para rangos, distinguir el estado de cada instancia con un sufijo de campo
(`fechaDesde`, `fechaHasta`), no reutilizar el mismo nombre. Ejemplo real con
2 rangos duplicados (Origen/Destino, 4 instancias en total, mas 2 combos de
Area, resuelto con un subcomponente parametrizado por nombre de campo en vez
de copiar el JSX):
`src/medi/CalidadGas/CopiaVariablesCalidad/components/containers/PanelContainers.jsx`
(componente `OrigenDestinoCard`).

## Props minimas verificadas

No es un listado exhaustivo; verificar la implementacion real antes de usar
una prop no documentada aqui.

| Prop | Tipo | Descripcion |
|---|---|---|
| value | Date / moment | Valor controlado |
| onChange | function | Nuevo valor (la forma exacta del callback difiere entre `CommonDatePicker` y `MediCommonDatePicker`; verificar cada implementacion) |
| label | string | Etiqueta visible |
| minDate / maxDate | Date | Limites de seleccion |
| disabled | boolean | Deshabilita el control |
| withArrows | boolean | Agrega flechas para avanzar/retroceder un dia |
| required | boolean | Marca el campo como obligatorio |

## Cuando NO usarlo

- No asumir que valida automaticamente que Desde sea anterior a Hasta en un
  rango: es logica a implementar en la pantalla (ver el ejemplo real de
  `CopiaVariablesCalidad`, que valida esto por separado).
- No usarlo para seleccionar una opcion de una lista (area, entidad, etc.):
  para eso ver [combo-autocomplete.md](./combo-autocomplete.md).
