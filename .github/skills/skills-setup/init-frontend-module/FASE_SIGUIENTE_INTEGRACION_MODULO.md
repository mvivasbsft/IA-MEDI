# Fase siguiente: creacion e integracion de modulos frontend

## 1. Objetivo

La skill crea la estructura fisica inicial de un modulo frontend TGS y puede integrarla funcionalmente en MEDI o SPAC.

## Estado actual (2026-10-05)

- El generador conserva la creacion de la estructura base y admite `-Integrate`.
- El `SKILL.md` guia la ubicacion simple o anidada y verifica el destino antes de ejecutar; el script repite la validacion y aborta si el directorio destino existe, incluso vacio. No existe `-Force`.
- Los roles los proporciona el usuario, uno o varios. No hay un numero fijo: una ruta puede necesitar dos, tres o mas roles.
- Todos los roles se validan contra exports existentes de `src/common/roles.js`. Ese archivo es de solo lectura para la skill y el script; si falta un rol, el usuario lo agrega manualmente y vuelve a ejecutar.
- Cada rol seleccionado se agrega al import de roles de `Module.jsx` y al array `roles={[...]}` de `RouteProtected`.
- El import del componente se agrega al final del bloque de imports: antes del comentario que precede a `TracedConsultaDePuntos` en MEDI y, en SPAC, antes de `class Module`.
- La integracion agrega una entrada de menu solo si `MenuKey` no existe. Si ya existe, conserva el valor sin verificar que coincida con `MenuLabel`.
- Los archivos compartidos se leen como UTF-8 y se conserva la presencia de BOM y el estilo de salto de linea. No es una deteccion de encodings arbitrarios.
- La ruta existente para el mismo componente se conserva sin reconciliar sus roles; una ruta asociada a otro componente provoca error. Los imports de componente se reemplazan en el ancla y los roles faltantes se agregan.
- No hay rollback: el modulo se crea antes de integrar, el menu se escribe antes que `Module.jsx` y un error posterior puede dejar modificaciones parciales. La ejecucion completa no es idempotente porque un destino existente hace abortar el script.
- La skill no ejecuta ESLint ni un formateador como parte de este flujo.
- La ejecucion previa del generador reescribio el archivo completo del menu y produjo mojibake en caracteres acentuados. La implementacion actual debe evitar repetirlo. La reparacion de texto ya corrompido es independiente y debe hacerse desde una copia confiable, no reescribiendo todo el menu como parte de la creacion de un modulo.
- El modulo de referencia presente en el workspace es `GasGenerator`, bajo `src/medi/Balance/Configuracion`, con ruta `/medi/balance/configuracion/gas-generator`, clave `MENU_MEDI.GASGENERATOR` y roles `ROLE_MEDI_VER_GASGENERATOR` y `ROLE_MEDI_EDITAR_GASGENERATOR`.
- La prueba real posterior creo `LineGas` en `src/medi/EFM/LineGas`, con ruta `/medi/efm/line-gas`, texto `EFM > Line Gas`, clave `MENU_MEDI.LINEGAS` y dos roles existentes validados. No se modifico `roles.js`.
- No se ejecutaron pruebas ni se crearon temporales, por instruccion del usuario. Se revisaron diagnosticos estaticos; esto no equivale a una validacion end-to-end del generador.

La tabla de alcance y las secciones siguientes describen el resultado funcional
esperado. Los puntos no incluidos en el estado actual anterior son pendientes,
no capacidades ya garantizadas por el Skill o el script. En particular, la
confirmacion resumida antes de modificar archivos compartidos, la resolucion
interactiva de conflictos de integracion y las validaciones posteriores siguen
siendo requisitos deseados, no pasos automatizados por el script.

El resultado esperado no es solamente una carpeta con archivos base. La skill debe dejar el modulo preparado para ser descubierto, protegido y ejecutado desde la aplicacion MEDI o SPAC.

La implementacion debe cubrir dos partes relacionadas, pero independientes:

1. **Creacion del modulo**: genera la estructura de carpetas y archivos.
2. **Integracion del modulo**: valida y reutiliza roles; registra menu, ruta, imports y `RouteProtected` en los archivos correspondientes.

La skill debe continuar siendo interactiva y realizar el proceso mediante preguntas, validaciones y confirmaciones explicitas.

---

## 2. Alcance funcional

Para cada modulo nuevo, la skill debe poder:

- Crear la carpeta fisica del modulo.
- Crear el componente principal `<ModuleName>.jsx`.
- Crear `Context.jsx`.
- Crear `Service.js`.
- Crear las carpetas `components/`, `components/buttons/`, `components/combos/`, `components/modals/`, `components/tables/` y `hooks/`.
- Calcular la ruta fisica completa del modulo.
- Calcular la ruta de React Router.
- Validar y reutilizar uno o varios roles existentes, sin editar el archivo de roles.
- Agregar la opcion correspondiente al menu.
- Agregar el import del componente en el modulo raiz de la categoria.
- Agregar el `RouteProtected` correspondiente.
- Usar el modulo de trazabilidad correcto (`MODULES.MEDI`, `MODULES.SPAC` u otra constante existente).
- Validar que no se dupliquen imports, roles, menu entries ni rutas.
- Pedir confirmacion antes de sobrescribir o modificar contenido existente.
- Preservar el encoding y las lineas no relacionadas de los archivos compartidos.

---

## 3. Conceptos y rutas

La skill debe diferenciar las siguientes rutas:

### 3.1. Ruta fisica del modulo

Es la ruta del filesystem calculada a partir de los parametros del generador:

```text
ProjectPath/FolderName/ModuleName
```

Ejemplo:

```text
src/medi/Balance/Configuracion/Protones
```

En el script esta ruta se representa mediante `$modDir`.

### 3.2. Ruta del componente

Es la ruta relativa utilizada por el import en `Module.jsx`:

```javascript
import Protones from "./Balance/Configuracion/Protones/Protones";
```

La ruta debe calcularse desde el archivo `src/medi/Module.jsx` o `src/spac/Module.jsx`, no desde la carpeta del modulo.

### 3.3. Ruta de React Router

Es la URL que activa la pantalla dentro de la aplicacion. No debe confundirse con `$modDir`.

Ejemplo:

```text
/medi/balance/configuracion/protones
```

La ruta debe ser un parametro confirmado por el usuario o una propuesta derivada de `FolderName` y `ModuleName`.

### 3.4. Categoria y modulo raiz

Para `Category = MEDI`, la integracion principal se realiza en:

```text
src/medi/Module.jsx
```

Para `Category = SPAC`, se debe identificar el modulo raiz equivalente de SPAC antes de modificar archivos.

La constante de trazabilidad existente se utiliza de esta forma:

```javascript
MODULES.MEDI
```

Cuando se mencione `MEDI_MODULE`, debe verificarse si se trata de una constante real del proyecto o de una forma conceptual de referirse al modulo raiz MEDI. La skill no debe inventar una constante que no exista.

---

## 4. Parametros de entrada

La skill debe recopilar, en este orden, los parametros faltantes:

### 4.1. Parametros actuales

- `ModuleName`: nombre del modulo y del componente principal.
- `Category`: `SPAC` o `MEDI`.
- `ProjectPath`: ruta base existente, normalmente `src/medi` o `src/spac`.
- `Anidamiento`: `Simple` o `Anidado`.
- `FolderName`: carpeta padre simple o ruta acumulada de carpetas.
- `Force`: no existe en el script actual; cualquier destino ya existente se rechaza.

### 4.2. Parametros de integracion

La siguiente fase debe incorporar estas decisiones:

- `RoutePath`: ruta web de la pantalla.
- `MenuLabel`: texto visible en el menu.
- `MenuKey`: nombre de la constante dentro de `MENU_MEDI` o `MENU_SPAC`.
- `Roles`: lista variable de roles existentes elegidos por el usuario.
- `ActivityModule`: modulo utilizado por `withActivityLog`, normalmente `MODULES.MEDI` o `MODULES.SPAC`.

Cuando sea posible, la skill puede proponer valores derivados, pero debe permitir que el usuario los confirme o modifique.

Ejemplo de valores propuestos para `Protones`:

```text
ModuleName: Protones
Category: MEDI
ProjectPath: src/medi
FolderName: Balance/Configuracion
RoutePath: /medi/balance/configuracion/protones
MenuKey: PROTONES
MenuLabel: Balance > Configuracion > Protones
Roles: ROLE_MEDI_VER_PROTONES, ROLE_MEDI_EDITAR_PROTONES
ActivityModule: MODULES.MEDI
```

---

## 5. Flujo interactivo esperado

### Paso 1: recopilar parametros basicos

La skill debe preguntar solamente los valores que no hayan sido proporcionados.

Orden:

1. Nombre del modulo.
2. Categoria.
3. Ruta base del proyecto.
4. Tipo de ubicacion: `Simple` o `Anidado`.
5. Resolucion guiada de `FolderName`.

La ruta base debe verificarse antes de continuar. Si no existe, el proceso debe detenerse.

### Paso 2: resolver carpetas

Para ubicaciones simples, no pedir una carpeta padre a ciegas:

1. Listar las carpetas existentes directamente bajo `ProjectPath`.
2. Mostrar cada carpeta existente como opcion seleccionable para `FolderName`.
3. Ofrecer tambien **Crear una carpeta nueva**. Si el usuario la elige,
   preguntar un nombre de un solo nivel y usarlo como `FolderName`.
4. Si no existen subcarpetas, ofrecer directamente crear una nueva.

Para ubicaciones anidadas:

1. Solicitar la carpeta raiz.
2. Verificar que exista.
3. Listar solamente subcarpetas reales.
4. Ofrecer:
  - `A`: usar la carpeta actual como destino final.
  - Numero de subcarpeta: entrar en ella.
  - `N`: crear una carpeta nueva en el nivel actual.
  - `M`: ingresar una ruta manual y validar sus segmentos.
  - `C`: cancelar.
5. Repetir hasta que el usuario elija una accion final.
6. Construir `FolderName` concatenando los niveles con `/`.

### Paso 3: recopilar integracion

Una vez conocida la ubicacion, preguntar o proponer:

1. Ruta web.
2. Texto visible del menu (`MenuLabel`).
3. Clave de menu (`MenuKey`), explicando para que se usa.
4. Solicitar un rol existente.
5. Preguntar si desea agregar otro rol; repetir hasta que indique que termino.
6. Confirmar el modulo de trazabilidad.

No inferir un conjunto fijo de roles por modo de acceso: la lista es definida
por el usuario y puede tener dos, tres o mas entradas.

La skill debe mostrar un resumen antes de modificar archivos compartidos.

#### MenuLabel, MenuKey y Module.jsx

Estos nombres cumplen funciones distintas y no son sinonimos:

- `MenuLabel` es el texto legible que identifica la pantalla en trazabilidad,
  por ejemplo `EFM > Line Gas`. Se guarda como valor de una propiedad dentro de
  `MENU_MEDI` en `src/common/constants/menu-module.jsx`.
- `MenuKey` es el identificador tecnico de esa propiedad, por ejemplo
  `LINEGAS`. La declaracion queda como `LINEGAS: 'EFM > Line Gas'` y se usa
  desde el modulo de rutas como `MENU_MEDI.LINEGAS`.
- `src/medi/Module.jsx` no es `menu-module.jsx`: importa el componente y declara
  el `RouteProtected`; alli se referencia `MENU_MEDI.LINEGAS` junto con
  `MODULES.MEDI` en `withActivityLog`.

La pregunta por `MenuKey` sirve para confirmar el nombre de la propiedad que se
va a agregar y luego referenciar desde `Module.jsx`. No crea un modulo adicional
ni una constante `MEDI_MODULE`.

### Paso 4: validar conflictos

Antes de ejecutar:

- Verificar si existe `ProjectPath/FolderName/ModuleName`.
- Verificar si tiene contenido.
- Verificar que cada rol seleccionado exista como export en `roles.js`.
- Verificar si ya existe `MenuKey`.
- Verificar si ya existe `RoutePath`.
- Verificar si ya existe el import del componente.
- Verificar si ya existe un `RouteProtected` para la misma ruta.

Si el destino existe, el Skill y el script abortan, tanto si tiene archivos
como si esta vacio. No se ofrece sobrescritura y no hay parametro `-Force`.

Si un rol no existe, detenerse e indicar que el usuario debe agregarlo
manualmente a `roles.js`; el generador nunca lo crea. En el script actual, una
clave de menu existente se conserva sin validar su texto; una ruta existente
para el mismo componente se conserva sin actualizarla y una ruta asignada a
otro componente provoca error. No hay un flujo interactivo para actualizar
esos registros.

#### Si el usuario cancela

Si responde **No** a la confirmacion final, se cancela el intento actual: no se
ejecuta el script ni se crean archivos del modulo ni se modifica el menu o
`Module.jsx`. La skill no vuelve automaticamente a la primera pregunta. Solo
reinicia el flujo si el usuario pide expresamente comenzar otra vez.

---

## 6. Fase A: creacion fisica del modulo

La creacion fisica debe continuar utilizando el script:

```text
.github/skills/init-frontend-module/scripts/create-tgs-module.ps1
```

Parametros:

```powershell
-ProjectPath
-Category
-FolderName
-ModuleName
```

No pasar `-Force`: el script no define ese parametro y no permite sobrescribir
un directorio destino existente.

La estructura esperada es:

```text
<ModuleName>/
    <ModuleName>.jsx
    Context.jsx
    Service.js
    components/
        buttons/
        combos/
        modals/
        tables/
    hooks/
```

La skill debe validar tambien las rutas relativas generadas en el componente y en `Service.js`, especialmente para modulos anidados. No debe asumir que una ruta fija como `../../../components` llega siempre a `src/components`.

---

## 7. Fase B: seleccionar y validar roles

Los roles se seleccionan del archivo existente:

```text
src/common/roles.js
```

El usuario puede indicar cualquier cantidad de roles (por ejemplo, dos roles de
visualizacion/edicion o tres permisos distintos). La nomenclatura puede servir
como referencia al usuario, pero el generador no propone crear ni crea roles.
Cada rol ingresado debe existir como export en `roles.js`.

Para MEDI, algunos nombres existentes siguen convenciones como:

```javascript
ROLE_MEDI_VER_<NOMBRE>
ROLE_MEDI_EDITAR_<NOMBRE>
```

Ejemplo:

```javascript
ROLE_MEDI_VER_PROTONES
ROLE_MEDI_EDITAR_PROTONES
```

Reglas obligatorias:

- `roles.js` es de solo lectura para la skill y el script.
- El usuario puede agregar manualmente un rol a `roles.js` antes de ejecutar la integracion.
- Si un nombre no existe como export, detenerse; no inventarlo ni escribirlo.
- No imponer que siempre haya un rol `VER` y otro `EDITAR`.
- Agregar todos los roles elegidos al import de roles del `Module.jsx` de categoria.
- Pasar todos los roles al array `roles={[...]}` de `RouteProtected`.

La skill informa los roles validados y reutilizados, y confirma que no modifico `roles.js`.

---

## 8. Fase C: menu

Para MEDI, el menu se registra en:

```text
src/common/constants/menu-module.jsx
```

Dentro de `MENU_MEDI` se debe agregar una constante como:

```javascript
PROTONES: 'Balance > Configuracion > Protones',
```

La clave debe coincidir con el valor usado en `Module.jsx`:

```javascript
MENU_MEDI.PROTONES
```

Reglas:

- No duplicar `MenuKey`.
- Agregar solamente la linea nueva necesaria dentro de `MENU_MEDI` o `MENU_SPAC`.
- No reescribir ni reformatear el archivo completo; preservar encoding, acentos y todas las lineas no relacionadas.
- Usar el texto confirmado por el usuario.
- Para SPAC, modificar el bloque de menu correspondiente, no `MENU_MEDI`.

---

## 9. Fase D: import del componente

En `src/medi/Module.jsx` se debe agregar el import del componente generado:

```javascript
import Protones from "./Balance/Configuracion/Protones/Protones";
```

El import debe:

- Apuntar al archivo real.
- Usar separadores `/`.
- Omitir la extension si ese es el estilo existente.
- Mantener el nombre del simbolo igual al nombre del modulo.
- Aparecer al final del bloque de imports, antes del comentario
  `// Se declara una sola vez fuera de renderModule` en MEDI. En SPAC, si no
  existe un comentario equivalente, ubicarlo inmediatamente antes de
  `class Module`.
- No duplicar un import existente.

La misma regla aplica al modulo raiz equivalente de SPAC.

---

## 10. Fase E: RouteProtected

En el `renderModule` correspondiente se debe agregar una ruta protegida:

```jsx
<RouteProtected
    exact
    path="/medi/balance/configuracion/protones"
    roles={[ROLE_VER_CALIDAD_GAS, ROLE_MEDI_VER_MAX_MIN_CALIDAD, ROLE_MEDI_EDITAR_VERIF_CALIDAD]}
    component={withActivityLog(Protones, MODULES.MEDI, MENU_MEDI.PROTONES)}
/>
```

La implementacion final debe respetar el formato predominante del archivo existente.

Reglas:

- Usar `exact` cuando la pantalla tenga una ruta unica.
- Usar todos los roles existentes confirmados por el usuario, sean dos, tres o mas.
- Usar `MODULES.MEDI` para pantallas MEDI, salvo que el proyecto tenga otra convencion comprobada.
- Usar el `MenuKey` correcto.
- No duplicar una ruta existente.
- Insertar la ruta dentro del bloque adecuado del modulo raiz.
- Verificar que el componente importado y los roles esten disponibles en el archivo.

---

## 11. MEDI_MODULE y trazabilidad

La skill debe revisar el concepto de `MEDI_MODULE` antes de modificar archivos.

En el codigo actual se observa la convencion:

```javascript
MODULES.MEDI
```

y el uso de trazabilidad:

```javascript
withActivityLog(Protones, MODULES.MEDI, MENU_MEDI.PROTONES)
```

Por lo tanto:

- Si `MEDI_MODULE` es un nombre funcional utilizado por el equipo, debe mapearse a la constante real existente.
- Si no existe una constante `MEDI_MODULE`, no se debe crear automaticamente sin confirmacion.
- La skill debe preferir `MODULES.MEDI` porque es la convencion comprobada en `Module.jsx`.
- Para SPAC debe utilizarse `MODULES.SPAC` y el menu de SPAC correspondiente.

---

## 12. Orden tecnico de modificaciones

El orden recomendado para evitar referencias rotas es:

1. Validar `ProjectPath`.
2. Resolver `FolderName`.
3. Calcular todas las rutas y nombres.
4. Verificar destino, conflictos y roles existentes.
5. Crear la estructura fisica.
6. Agregar una unica entrada de menu si no existe.
7. Agregar todos los roles al import existente de roles en el modulo raiz.
8. Agregar el import del componente despues del import de roles.
9. Agregar `RouteProtected` con todos los roles elegidos.
10. Validar rutas y simbolos.
11. Reportar el resultado completo.

Si una etapa falla, la skill debe informar que etapas se completaron y cuales no, sin ocultar modificaciones parciales.

---

## 13. Confirmacion final antes de modificar archivos compartidos

Antes de modificar `menu-module.jsx` o `Module.jsx`, mostrar un resumen similar a:

```text
Se creara el modulo:
  src/medi/Balance/Configuracion/Protones

Se usaran los roles existentes:
  - ROLE_MEDI_VER_PROTONES
  - ROLE_MEDI_EDITAR_PROTONES
  (validados en src/common/roles.js; este archivo no se modifica)

Se agregara:
  - MENU_MEDI.PROTONES
  - import Protones en src/medi/Module.jsx
  - RouteProtected /medi/balance/configuracion/protones
  - trazabilidad MODULES.MEDI

Archivos a modificar:
  - src/common/constants/menu-module.jsx
  - src/medi/Module.jsx

¿Confirmas estos cambios? Si / No
```

Sin confirmacion, la skill no debe modificar archivos compartidos.

---

## 14. Validaciones posteriores

Despues de ejecutar la integracion, la skill debe comprobar:

- El directorio del modulo existe.
- El componente principal existe.
- El import apunta a un archivo existente.
- Los roles usados por `RouteProtected` estan definidos.
- `MenuKey` esta definido.
- `RoutePath` aparece una sola vez.
- El `RouteProtected` referencia el componente correcto.
- `MODULES.MEDI` o `MODULES.SPAC` es valido.
- No se generaron imports duplicados.
- No se generaron constantes duplicadas.
- No hay errores de sintaxis o diagnosticos en los archivos modificados.

Si el proyecto tiene un comando de lint, typecheck, build o test especifico, debe ejecutarse el chequeo mas acotado disponible y reportarse su resultado.

---

## 15. Reporte final esperado

El reporte final debe indicar:

- Ruta fisica creada.
- Tipo de ubicacion: simple o anidada.
- Carpetas intermedias creadas.
- Categoria.
- Ruta web.
- Menu agregado o reutilizado.
- Roles existentes validados y reutilizados; indicar que `roles.js` no se modifico.
- Archivo de modulo raiz actualizado.
- `RouteProtected` agregado.
- Modulo de trazabilidad utilizado.
  - Cualquier modificacion parcial si una etapa de integracion fallo; el script no hace rollback.
- Validaciones ejecutadas y resultado.
- Cualquier limitacion o paso manual pendiente.

Ejemplo:

```text
Modulo creado:
  src/medi/Balance/Configuracion/Protones

Integracion:
  - Ruta: /medi/balance/configuracion/protones
  - Menu: MENU_MEDI.PROTONES
  - Roles validados: ROLE_MEDI_VER_PROTONES, ROLE_MEDI_EDITAR_PROTONES
  - Trazabilidad: MODULES.MEDI
  - RouteProtected agregado en src/medi/Module.jsx

Carpetas nuevas:
  Ninguna

Validacion:
  Sin errores detectados.
```

---

## 16. Principio de seguridad

### Aprobacion para ejecutar PowerShell

El script actual no contiene `Read-Host`, `ShouldProcess`, `ConfirmImpact` ni
`-Confirm`; tampoco muestra una pregunta interactiva de seguridad. La
confirmacion de la skill antes de ejecutarlo es un paso conversacional separado.

Si aparece un dialogo de aprobacion al lanzar el comando, puede ser la
autorizacion de ejecucion de VS Code o de la herramienta que inicia el terminal,
no una pregunta generada por `create-tgs-module.ps1`. Si el propio PowerShell
muestra un mensaje de politica de ejecucion, editor no confiable o antivirus,
hay que leer el texto exacto para identificar el origen; este documento no debe
afirmar que esos avisos provienen de la skill.

El script no define `-Force`: si la carpeta destino ya existe, aborta sin
sobrescribirla, aun cuando este vacia. La autorizacion de ejecucion de PowerShell
o VS Code, si aparece, es independiente de esta validacion de colision.

La skill debe ser automatica en la creacion, pero conservadora al modificar archivos compartidos.

Objetivo de seguridad para la evolucion del flujo:

- Sobrescribir un modulo con contenido sin confirmacion.
- Duplicar roles, menu entries, imports o rutas.
- Crear una constante `MEDI_MODULE` sin verificar la convencion existente.
- Suponer que una ruta fisica es igual a una ruta web.
- Modificar archivos no relacionados con la integracion solicitada.
- Continuar silenciosamente despues de una modificacion parcial fallida.
