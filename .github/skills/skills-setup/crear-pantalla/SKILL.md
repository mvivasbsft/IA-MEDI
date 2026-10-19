---
name: crear-pantalla
description: 'Construye una pantalla piloto MEDI sobre un modulo ya inicializado, componiendo contenedores, tabla, acciones y modal con componentes existentes y datos mock asincronos. Usar cuando el dev invoque /crear-pantalla para implementar un requerimiento funcional.'
argument-hint: 'Ruta del modulo existente y requerimiento funcional .md'
user-invocable: true
disable-model-invocation: true
---

# Crear pantalla: piloto MEDI

## Alcance

Construir y conectar componentes locales de una pantalla existente usando
referencias verificadas y un servicio mock que devuelve una promesa con datos.
No basta con crear carpetas o contenedores vacios.

El dev invoca primero `init-frontend-module` cuando desea inicializar el
modulo. Luego decide cuando invocar esta skill. No ejecutar esa otra skill,
su script ni volver a inicializar el destino. No elegir automaticamente el
ultimo modulo creado.

## 1. Resolver destino y requerimiento

Preguntar solo lo que falte, en este orden:

1. Sobre que modulo/pantalla existente bajo `src/medi` se trabajara. Permitir
   indicar una ruta o navegar carpetas existentes nivel por nivel hasta elegir
   el modulo. No crear otra ubicacion ni una carpeta fija de pruebas.
2. Que requerimiento funcional implementar: archivo Markdown, adjunto o texto
   del dev. No solicitar acceso a SharePoint/Jira en este piloto.
3. Que datos de ejemplo usar. Si no fueron proporcionados, proponer un array
   coherente con el requerimiento para que el dev lo confirme.

Para resolver el destino cuando el dev no dio una ruta concreta, enumerar las
carpetas hijas inmediatas existentes de `src/medi` y mostrar sus nombres como
opciones seleccionables en `ask_user`. No usar una busqueda limitada a archivos
del primer nivel como si fuera el listado de modulos: puede ocultar carpetas
anidadas. Despues de cada eleccion, enumerar las carpetas hijas inmediatas de
la carpeta elegida y ofrecerlas como opciones; repetir nivel por nivel hasta
identificar el modulo/pantalla. No pedir una ruta abierta mientras haya
opciones que se puedan presentar, no dejar vacia la lista de opciones y no
elegir automaticamente un destino. Si el dev ya indico una ruta concreta,
verificarla directamente en lugar de iniciar la navegacion.

Comprobar que el destino este dentro de `src/medi` y existan componente
principal, contexto y servicio. Si falta la base, informar y detenerse para
que el dev la resuelva; no ejecutar el inicializador por su cuenta.

Inspeccionar los archivos y usos vecinos necesarios. Distinguir estructura
generada y trabajo ya implementado. Un directorio existente es esperado:
completar archivos mediante cambios acotados, no reemplazar trabajo del dev.
Si hay componentes ya creados, proponer reutilizarlos antes de duplicarlos.

Las imagenes adjuntas pueden complementar el requerimiento si el modelo
admite vision; preguntar por reglas o detalles que no sean visibles.

## 2. Elegir las piezas y leer sus referencias

Leer solo los recursos pertinentes, antes de implementar la pieza:

| Necesidad | Recurso |
|-----------|---------|
| Panel de filtros y controles | [Panel Container](./references/componentes/panel-container/panel-container.md) |
| Tabla y adaptacion de registros | [MediCommonTable](./references/componentes/tabla.md) |
| Barra de acciones local | [Action Bar](./references/componentes/action-bar.md) |
| Dialogo con contenido propio | [Modal](./references/componentes/modal.md) |
| Datos y contrato asincrono del piloto | [Servicio mock](./references/servicio-mock.md) |

Estas son referencias minimas para el piloto, no un catalogo completo. Para
selectores, fechas u otra variante no documentada, verificar la implementacion
real y un uso cercano antes de elegir props; no inventar APIs. No copiar logica
de negocio de una pantalla ajena. Importar componentes comunes, no clonarlos.

## 3. Resolver dudas y confirmar

Extraer del requerimiento los campos, columnas, acciones y estados. Preguntar
solo por ambiguedades imprescindibles, por bloques y paso a paso. Para la
tabla, confirmar nombres/keys/tipos y datos. No preguntar por seleccion o
paginacion si no se solicitaron; declararlas omitidas en la confirmacion.
No exigir un endpoint real: este piloto utiliza un mock local.

Aplicar estos defaults antes de preguntar:

- No asumir `TextField` para todos los campos ni agregar un control "Limpiar
   filtros": no son patrones verificados del proyecto (ver
   [panel-container.md](./references/componentes/panel-container/panel-container.md#resolver-controles-de-forma-simple-base-de-crear-pantalla)).
   En esta fase (base simple de `/crear-pantalla`) alcanza con pedirle al dev,
   en un solo bloque, un roster minimo: cuantos controles tiene el panel y
   para cada uno su nombre y naturaleza general (texto libre / fecha / combo
   selector / boton Listar). No profundizar aqui en props exactas,
   validaciones o casos borde por campo: esa precision queda para una skill
   especifica de Panel Container que se invocara mas adelante sobre esta
   misma base, leyendo lo ya construido en vez de decidir el contrato de
   nuevo. Para elegir familia de componente (`Common*`, `Medi*` o `Spac*`)
   revisar 1-2 `PanelContainer.jsx` reales del mismo modulo y replicar lo que
   usan; no asumirla solo por el nombre del modulo ni por la sola existencia
   de un archivo con ese prefijo (ej. existe `MediCommonDialog` pero varias
   pantallas MEDI reales usan `CommonDialog`). Esto no reemplaza ni evita la
   interview obligatoria de
   [Resolver la distribucion espacial (Grid)](./references/componentes/panel-container/panel-container.md#resolver-la-distribucion-espacial-grid)
   del Panel Container, que exige preguntar siempre que haya 2 o mas controles
   (los del roster cuentan como controles), aunque esta pieza se confirme
   junto con otras en el mismo mensaje.
- Preguntar, antes de montar las piezas en el componente principal, si el
   `PanelContainer` debe quedar como hijo de
   `CommonHeader`/`MediCommonHeader` o como hermano fuera de el. Esta decision
   aplica solo al `PanelContainer`: la tabla, la barra de acciones y el modal
   quedan como hermanos fuera del header. No asumir ninguna de las dos
   opciones para el panel ni copiar la disposicion del scaffold generado por
   `init-frontend-module` sin confirmarla con el dev.
- No agregar seleccion ni paginacion si no se solicitaron y la pantalla no las
   necesita para cumplir el requerimiento. Declarar su omision en el resumen.
- Preguntar si falta una regla que cambie el resultado funcional, por ejemplo
   rangos de fecha, inclusividad, normalizacion especial o filtro remoto. No
   preguntar por decisiones visuales reversibles cuando existe un default local.

Antes de crear componentes, asignar una unica responsabilidad a cada archivo.
Si se usan ambos, `TableContainer` consume el contexto y presenta carga, error
o vacio, y conecta callbacks; la tabla local define columnas, adapta celdas y
renderiza `MediCommonTable`. No implementar ni pegar dos variantes del mismo
componente. Si un archivo ya existe, modificarlo en lugar de volver a agregar
una segunda implementacion.

Presentar para confirmacion:

- Ruta del modulo y resumen de comportamiento.
- Contenedores y componentes a crear o reutilizar.
- Archivos existentes a completar (principal, contexto, servicio).
- Array de ejemplo y funcion de servicio que simula el GET sin HTTP.
- Acciones esperadas, alcance local y comprobaciones de funcionamiento.

Esperar aprobacion antes de escribir. Si el dev cancela, no editar ni reiniciar
el flujo. No agregar funcionalidades no acordadas solo para cubrir el catalogo.

Esta base prioriza quedar simple y coherente con componentes reales del
proyecto; no agota casos borde ni validaciones finas de cada control. Esa
profundizacion se resuelve mas adelante con una skill dedicada por
contenedor (Panel Container, Tabla, Action Bar, Modal) que leera esta base
ya construida antes de preguntar, en vez de volver a decidir el contrato
desde cero.

## 4. Implementar incrementalmente

1. Completar el servicio local con una operacion asincrona que devuelva el
   array mock confirmado y respete los filtros acordados. No inventar endpoints
   ni realizar HTTP. Mantener intactas operaciones existentes ajenas al piloto.
   Si el Panel Container usa selectores con datos propios (combos cuyas
   opciones vienen de su propio mock), agregar una funcion asincrona separada
   por cada uno; ver `./references/componentes/panel-container/controles/combo-autocomplete.md`.
2. Completar el contexto o hook local: datos, carga, error, acciones y estado
   compartido necesarios. Consumir la promesa con manejo de error y finalizacion
   de carga; no introducir una segunda fuente contradictoria de estado.
   El mecanismo interno (useState, useReducer u otro) queda a criterio del
   agente segun el caso y las convenciones del modulo existente.
   <!-- PENDIENTE: definir un criterio explicito del repo para elegir entre
   useState y useReducer (u otro mecanismo) al completar el contexto de
   pantalla. Por ahora es una decision abierta del agente, no una regla fija. -->
3. Crear el Panel Container como un maquetado interactivo: el valor de cada
   control se registra en el contexto compartido (no en estado local aislado
   del propio Panel Container), los selectores con datos propios (combo/
   autocomplete) se cargan con su array mock de opciones, y el boton final
   arma un payload con los valores actuales sin llamar todavia a
   `Service.getRegistros()` ni disparar ninguna consulta real; ver
   [Maquetado interactivo sin logica de negocio](./references/componentes/panel-container/panel-container.md#maquetado-interactivo-sin-logica-de-negocio-base-de-crear-pantalla).
4. Crear Table Container y tabla local si se necesitan ambos; adaptar los
   registros de dominio al contrato de la tabla comun elegida.
5. Crear Action Bar y Modal locales cuando el requerimiento los necesite.
   Conectar habilitacion, apertura, aceptar y cancelar. Una edicion mock solo
   modifica estado local; no afirmar persistencia en backend. Para campos
   controlados, seguir el patron de la referencia del Modal y capturar el
   valor del evento antes de llamar a un actualizador funcional.
6. Conectar las piezas al componente principal bajo su proveedor existente.
   Preservar props/contexto recibidos desde la aplicacion y convenciones locales.

Despues de cada cambio significativo, ejecutar la comprobacion acotada
disponible antes de seguir. No dejar imports rotos entre pasos. Reutilizar
helpers y estilos existentes; calcular imports desde cada archivo real.
Crear solo carpetas y abstracciones que la pantalla necesite.

Al terminar cada archivo editado, comprobar que tenga una sola declaracion del
componente esperado, un solo `export default`, imports al inicio y ningun
bloque duplicado o concatenado despues del export. Ejecutar diagnosticos del
archivo antes de editar el siguiente; corregir ahi mismo cualquier error.

## 5. Verificar y reportar

- Validar imports, sintaxis y diagnosticos de archivos tocados con las
  herramientas y comandos disponibles del repo. No instalar dependencias
  adicionales para este piloto sin justificar y confirmar la necesidad.
- Antes de ejecutar un comando en terminal, explicar el comando exacto, el shell,
   que valida y si crea o modifica archivos. Por ejemplo, `npm run build-js`
   compila la app y genera artefactos en `build/`; no es una prueba visual ni un
   despliegue. Pedir aprobacion antes de comandos que escriban artefactos o
   levanten procesos. Si el dev omite una comprobacion, no volver a solicitarla:
   continuar con diagnosticos disponibles y reportar que quedo sin verificar.
- Preguntar si ya hay un `npm start` corriendo desde antes de esta sesion de
  cambios. Esta skill crea varios archivos interdependientes en pasos
  separados (hook, containers, modal); con un dev server activo y observando
  `src/` en vivo, `react-scripts` (CRA 2.1.1) puede disparar una recompilacion
  en caliente a mitad de esa secuencia y arrojar un error transitorio de
  modulo no encontrado (por ejemplo `ENOENT ... open '...useXContext.jsx'`,
  aunque el archivo real creado sea `.js` y el contenido sea correcto: webpack
  termina mostrando la ultima extension que prueba en su resolucion, no la
  causa real). Si aparece un error asi al terminar, la correccion verificada
  es reiniciar el dev server desde cero (cerrarlo y volver a correr
  `npm start`), no modificar el codigo generado. Si el dev ya tiene un server
  activo, avisar este riesgo antes de implementar y sugerir reiniciarlo al
  finalizar en vez de confiar en el hot reload de la sesion en curso.
- Verificar que los registros mock llegan del servicio al contexto y a la
  tabla, y que carga, vacio y errores tienen un tratamiento apropiado.
- Probar las interacciones acordadas con tests existentes o en navegador si
  la pantalla ya tiene una ruta y el ambiente lo permite. Aceptar/cancelar
   deben producir los efectos establecidos por el requerimiento. En el modal,
   escribir varios caracteres en al menos un campo y comprobar que el valor
   permanece antes de aceptar o cancelar.
- Si no hay ruta integrada, acordar una prueba de renderizado o reportar
  explicitamente la falta de validacion visual. No crear rutas temporales.
- Informar archivos creados/modificados, componentes comunes reutilizados,
  datos mock, comprobaciones ejecutadas y pasos no verificados. No confundir
  compilacion con prueba de interaccion ni mock con un GET real.

## Limites

No modificar `init-frontend-module`, componentes compartidos, `roles.js`, menu
ni el modulo raiz de rutas. Conservar la integracion previa del modulo si
existe. No obtener documentos de servicios externos ni implementar backend.
No ejecutar todas las fases futuras: este alcance prueba composicion local.