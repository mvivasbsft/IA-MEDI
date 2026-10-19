# Panel Container

## Que es y para que sirve

El Panel Container es la superficie de entrada de la pantalla. No muestra los
registros del listado (eso es responsabilidad exclusiva de la Tabla, un
componente aparte): administra los valores que el usuario ingresa o
selecciona y los envia como parametros a una operacion del Service (mockeada
en este piloto) cuando se dispara la accion principal, normalmente Listar.
El resultado de esa llamada se guarda en el contexto compartido; la Tabla lo
consume por separado. No existe un rol de "solo mostrar datos" dentro del
panel mismo.

Patron de contenedor local, no componente global para importar. Crear la
pieza de la pantalla que compone controles y conecta sus acciones al
contexto/hook. Ejemplo real:
`src/medi/CalidadGas/DatosCalidadGas/components/containers/PanelContainer.jsx`.
Ese ejemplo usa `MediCommonAC`, `CommonDatePicker`, `CommonListarButton` y
contexto propio; no copiar sus reglas de negocio al piloto.

## Roles de un control dentro del panel

Todo control del panel cumple uno o mas de estos roles (no son excluyentes
entre si):

| Rol | Que hace | Ejemplo real | Tiene su propio mock/GET |
|---|---|---|---|
| Filtro local post-fetch | Recorta datos ya traidos, sin volver a pedirle nada al servicio | Patron minoritario en el repo; confirmar con el dev antes de asumirlo (no hay ejemplo real estable para citar) | No |
| Parametro de consulta pre-fetch | Su valor se envia como argumento al service cuando se dispara Listar; decide que se trae, no recorta nada visible | `selectedDate` en `src/medi/Reportes/ConsultaAguaAzufreMetanol/Context.jsx` (`Service.listar(selectedDate)`) | No necesariamente |
| Selector con datos propios | Sus opciones (no su valor elegido) vienen de un mock/GET separado del listado principal | Combo Area en `src/medi/CalidadGas/CopiaVariablesCalidad/hooks/useAreas.js` (`Service.getAreasCalidad()`) | Si |

Un mismo control puede combinar roles: un combo puede tener datos propios
(opciones via mock) y ademas actuar como parametro de consulta (su valor
elegido se manda al listar). Ver el detalle de cada tipo de control en
[Piezas disponibles](#piezas-disponibles-controles) mas abajo.

## Piezas disponibles (controles/)

No son una API universal ni un catalogo cerrado; leer solo la ficha
pertinente al requerimiento, no todas:

| Necesidad | Ficha |
|-----------|-------|
| Filtro de texto simple sobre datos ya traidos | [Filtro de texto](./controles/filtro-texto.md) |
| Fecha puntual o rango de fechas | [Selector de fecha](./controles/selector-fecha.md) |
| Combo / autocomplete, con o sin datos propios | [Combo / Autocomplete](./controles/combo-autocomplete.md) |
| Boton que dispara la consulta principal | [Boton Listar](./controles/boton-listar.md) |

Para un control no documentado aqui, verificar la implementacion real y un
uso cercano antes de elegir props; no inventar APIs.

## Resolver el roster de controles antes de implementar

Antes de escribir codigo, armar y confirmar con el dev una tabla con un
registro por control (no por columna de tabla: puede haber controles sin
columna asociada, como un rango de fechas que filtra pero no se muestra):

| # | Concepto (campo del requerimiento) | Tipo de control | Cantidad | Rol | Origen de datos |
|---|---|---|---|---|---|
| 1 | Area | Combo (`MediCommonAC`) | 1 | Selector con datos propios + parametro de consulta | Mock local (ej. `Service.getAreas()`) |
| 2 | Fecha | Selector de fecha | 1 | Parametro de consulta | Ninguno (valor de estado local) |
| 3 | PM | Filtro de texto | 1 | Filtro local post-fetch | Ninguno (ya viene en los datos listados) |
| 4 | Listar | Boton | 1 | Accion — dispara el service principal | Mock local (ej. `Service.getRegistros()`) |

Es un ejemplo ilustrativo, no una obligacion de usar esas filas. Para este
piloto, la columna "Origen de datos" siempre resuelve a una funcion
asincrona local sin HTTP (ver [Servicio mock](../../servicio-mock.md)). Un
mismo `Service.js` puede necesitar mas de una funcion mock: una para el
listado principal y una por cada selector con datos propios.

## Resolver la distribucion espacial (Grid)

**Regla obligatoria, sin excepcion:** esta interview no se omite ni se
reemplaza por el criterio simple de controles (seccion
[Resolver controles de forma simple](#resolver-controles-de-forma-simple-base-de-crear-pantalla)
mas abajo). Ese otro criterio resuelve unicamente que controles existen y de
que familia; nunca resuelve cuantos controles hay, como se agrupan o
como se distribuyen en Grid. Si el panel tiene 2 o mas controles, preguntar
siempre antes de escribir codigo, aunque el resto de las piezas del piloto
(tabla, action bar, modal) se este confirmando o ya se haya confirmado en el
mismo mensaje: no se puede dar por buena una distribucion de Grid que el dev
no eligio explicitamente.

Cuando preguntar por la distribucion de los controles en filas y columnas,
y cuando aplicar un default sin preguntar:

| Cantidad de filtros/controles | Hay boton Listar en el panel | Que hacer |
|---|---|---|
| 0 | Si | Solo el boton, solo en la fila. |
| 1 | Si o No | Nada que distribuir entre controles. Si hay boton, aplicar el default de 2 zonas (control + boton a la derecha) sin preguntar. |
| 1 | No | Un unico control en una fila, sin zonas. |
| 2 o mas | Si o No | Preguntar como se agrupan/ordenan entre si (una fila en orden simple, o agrupados con ancho particular). |
| cualquiera | — | Si el dev pide explicitamente mas de una fila o agrupamiento en cards, resolver con la interview completa de filas/columnas (ver mas abajo), sin importar la cantidad de controles. |

Default verificado en 37 pantallas de `src/medi/**/components/containers/PanelContainer.jsx`:
una fila con dos zonas, controles a la izquierda y el boton Listar alineado
a la derecha. Ejemplo real:
`src/medi/Reportes/ConsultaAguaAzufreMetanol/components/containers/PanelContainer.jsx`
(`Grid item` ancho 9/12 para los controles, `Grid item` ancho 3/12 con
`justifyContent: "flex-end"` para el boton). Presentar este default como
supuesto en la confirmacion, no como pregunta bloqueante. **Este default solo
aplica a la fila "1 control + boton" de la tabla anterior; con 2 o mas
controles no corresponde y la pregunta de agrupamiento/orden es obligatoria.**

Si el requerimiento pide mas de una fila o agrupamiento (ej. tarjetas
Origen/Destino como en `src/medi/CalidadGas/CopiaVariablesCalidad`), resolver
con una interview por bloques:

1. Preguntar cantidad de filas.
2. Por cada fila, preguntar cuantas columnas tiene, que control del roster
   va en cada una (en orden) y el ancho relativo entre columnas (ofrecer
   presets simples: "ancho igual entre todas", "una columna mas ancha que el
   resto", "una columna ocupa toda la fila"; traducir a fracciones de 12).
3. El dev tambien puede adjuntar una imagen de referencia si el modelo
   admite vision. El agente propone una interpretacion (filas, columnas,
   proporciones aproximadas) pero no implementa directo desde la imagen: el
   tipo exacto de cada control (texto libre vs combo cerrado, fecha puntual
   vs rango) no se puede inferir con certeza de una imagen y se pregunta
   aparte.
4. Presentar siempre el resultado como una tabla fila/columna/control/ancho
   para confirmar antes de implementar, enlazando cada celda al # del roster
   de controles.

## Ejemplo minimo

```jsx
import React from 'react';
import { Button, Grid } from '@material-ui/core';

const PanelContainer = ({ onListar, loading }) => (
    <Grid container>
        <Button onClick={onListar} disabled={loading}>Listar</Button>
    </Grid>
);

export default PanelContainer;
```

Este ejemplo ilustra el patron; los controles reales y sus componentes se
eligen leyendo el requerimiento, el roster confirmado y verificando sus
implementaciones.

## Resolver controles de forma simple (base de /crear-pantalla)

No asumir `TextField` para cada campo de datos ni agregar un control para
limpiar filtros por default: relevado contra 48 `PanelContainer.jsx` reales
de MEDI, ninguno tiene un control "Limpiar filtros" y el 77% si tiene boton
Listar. Ese patron de "solo filtros de texto, sin Listar" no es una
convencion del proyecto; no usarlo como default silencioso.

En esta fase mantener la pregunta simple: pedir al dev un roster minimo
(cantidad de controles, nombre y naturaleza general -- texto libre / fecha /
combo selector / boton Listar) sin bajar todavia a props exactas o reglas de
validacion por campo; eso se resuelve despues, con una skill dedicada por
contenedor que lea esta base ya construida. Para la familia de componente
(`Common*`, `Medi*` o `Spac*` segun el modulo) revisar 1-2 `PanelContainer.jsx`
reales cercanos y replicar lo que usan, sin asumirlo solo por el prefijo del
modulo.

Esto resuelve unicamente que controles existen y de que familia; no decide
en que orden van ni como se distribuyen en Grid. Con 2 o mas controles (los
del roster cuentan como controles), la distribucion sigue siendo una
pregunta obligatoria: ver
[Resolver la distribucion espacial (Grid)](#resolver-la-distribucion-espacial-grid).
No usar este criterio simple como justificacion para omitir esa pregunta.

## Maquetado interactivo sin logica de negocio (base de /crear-pantalla)

En esta fase, el Panel Container se arma como un maquetado interactivo, no
una cascara estatica ni la logica final de negocio:

- El valor de cada control (combo, fecha, texto) se registra en el
  `Context`/reducer compartido de la pantalla, no en estado local aislado del
  propio `PanelContainer`, aunque todavia no dispare ninguna consulta real.
  Asi, cuando una skill futura por contenedor conecte la logica de negocio,
  ya encuentra el lugar correcto donde engancharla en vez de tener que migrar
  el estado de un lugar a otro.
- Los selectores con datos propios (combo/autocomplete) se cargan con un
  array mock de opciones para que el dev pueda interactuar de verdad con el
  control (ver [combo-autocomplete.md](./controles/combo-autocomplete.md)).
  Los controles de fecha o texto libre no necesitan datos mock propios: les
  alcanza con su valor controlado en el Context.
- El boton final del roster (`CommonListarButton`/`MediCommonListarButton` u
  otro acordado) arma un payload con los valores actuales de todos los
  controles del panel y lo deja disponible (por ejemplo en el propio estado
  del Context), pero no llama a `Service.getRegistros()` ni dispara ninguna
  consulta real todavia. Esa conexion queda para la skill futura por
  contenedor.

## Composicion interna (no necesitas consultarlos)

- `Grid` de Material UI: distribucion de los controles del ejemplo minimo.

## Componentes relacionados (consulta estas referencias)

- [Tabla](../tabla.md): recibe el resultado de la consulta iniciada desde el panel.
- [Modal](../modal.md): si la accion requiere confirmacion o presenta un dialogo.

## Cuando NO usarlo

- No importar un PanelContainer de otra pantalla que dependa de su contexto.
- No incluir operaciones HTTP o configuracion de columnas solo por proximidad;
  mantener responsabilidades en servicio y tabla locales.