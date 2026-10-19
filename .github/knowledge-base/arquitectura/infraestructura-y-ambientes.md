# Infraestructura y ambientes

- **Origen:** [TGS013 - Infraestructura y Ambientes — Confluence](https://baufest.atlassian.net/wiki/spaces/TGS400/pages/3515941450/TGS013+-+Infraestructura+y+Ambientes)
- **Fecha de importación:** 2026-10-19
- **Método de importación:** contenido proporcionado por el usuario; no se consultó Confluence ni se realizaron conexiones.
- **Versión y última modificación en Confluence:** no informadas.
- **Estado de la importación:** versión saneada; correspondencias de la tabla original pendientes de confirmar.

## Notas de conversión y confidencialidad

El texto recibido contenía una tabla sin separadores y credenciales de acceso.
Este documento no conserva contraseñas, identificadores de cuentas ni la tabla
de usuarios. No se reconstruyeron asociaciones entre cuentas, módulos o roles.
Tampoco se copiaron direcciones IP privadas ni cadenas de conexión internas:
deben consultarse en el inventario de infraestructura autorizado del equipo.
Estas omisiones son intencionales; esta no es una copia íntegra del original.

Las descripciones funcionales permiten identificar Local, DEV, SIT, UAT, UAT2
y PROD. La tabla también menciona UPG y Vaca Muerta, pero no permite asignar
con seguridad todos sus campos. No se infirieron esas correspondencias.
Las versiones y responsabilidades se documentan según la fuente recibida,
sin verificar su vigencia.

## Introducción

El presente documento describe la infraestructura, la composición de los
diferentes ambientes de la aplicación eTGS, su propósito y su correcto uso.

## Ambientes

| Ambiente | Propósito | Base de datos según la descripción | Gestión indicada en la descripción |
|---|---|---|---|
| Local | Desarrollo y pruebas preliminares del desarrollador. | PDBDEV | Desarrollador |
| DEV | Pruebas del equipo de desarrollo de Baufest en un ambiente desplegado. | PDBDEV por defecto | Equipo de desarrollo; responsabilidades operativas a confirmar |
| SIT | QA interno de Baufest. | PDBSIT por defecto | El equipo QA realiza los despliegues |
| UAT | Pruebas del cliente sobre las entregas. | PDBQUA por defecto | Cliente; detalle operativo a confirmar |
| UAT2 | Ambiente auxiliar con acceso directo o mediante balanceador. | Configurable para emular DEV, SIT o UAT | TGS configura el balanceador; la base se cambia mediante un job de Jenkins |
| PROD | Producción, con varias instancias detrás de un balanceador. | Correspondencia de la tabla pendiente de confirmar | Infraestructura y despliegue productivo bajo responsabilidad del cliente |
| UPG | Mencionado en la tabla original; propósito no descrito. | Pendiente de confirmar | Pendiente de confirmar |
| Vaca Muerta | Mencionado en la tabla original; propósito no descrito. | Pendiente de confirmar | Pendiente de confirmar |

### Local

Es el ambiente de desarrollo por defecto, donde se realizan los cambios y las
pruebas preliminares.

El backend se inicia con el servidor Tomcat embebido de Spring Boot y queda
disponible para consumo por API o por la interfaz de usuario. El frontend se
inicia por separado mediante el servidor de desarrollo de React.

| Servicio local | URL |
|---|---|
| Interfaz de usuario | `http://localhost:3000/etgs` |
| API del backend | `http://localhost:8080/etgs/api` |

Ambos procedimientos se describen en el documento de Confluence
«TGS013 - Setup de Ambiente de Trabajo», cuyo enlace no fue proporcionado.
La base de datos de pruebas indicada para Local es PDBDEV.

### Desarrollo (DEV)

El equipo de desarrollo de Baufest utiliza DEV para realizar pruebas en un
ambiente desplegado. Según el documento, desplegar allí reduce las diferencias
entre las condiciones de desarrollo local y las de SIT o PROD.

La base por defecto es PDBDEV. Esta base es compartida con otras consultoras,
por lo que puede haber inconsistencias durante las pruebas. No debe asumirse
que los datos o las operaciones están aislados.

### QA interno (SIT)

El equipo de QA de Baufest utiliza SIT para las pruebas pertinentes y tiene
a su cargo los despliegues en este ambiente. Su base por defecto es PDBSIT.

### QA usuario (UAT)

El cliente utiliza UAT para realizar pruebas sobre las entregas. El documento
lo identifica como el límite del flujo de entregas del equipo. Su base por
defecto es PDBQUA.

### Auxiliar con balanceador (UAT2)

Permite probar tanto el acceso directo a una instancia como el acceso mediante
un balanceador. Según la fuente, el balanceador actúa también como filtro de
IPs y solicitudes y su configuración corresponde a TGS.

La base de datos es configurable mediante un job de Jenkins para emular DEV,
SIT o UAT. Antes de realizar una prueba debe confirmarse la base activa;
el nombre del ambiente por sí solo no determina los datos afectados.

### Producción (PROD)

La fuente indica que hay poca información disponible y que existen varias
instancias detrás de un balanceador. La infraestructura y el proceso de
despliegue a producción son responsabilidad del cliente.

Este documento no autoriza accesos, pruebas ni modificaciones en producción.

### Inventario de acceso y tabla original

La tabla original incluye owners, managers, URLs directas, balanceadores,
servidores, bases y URLs de la aplicación con framework anterior. Para los
servidores desplegados menciona JBoss EAP 7.4; para Local, Tomcat/Node embebido.
La asignación exacta de todos los campos a los ocho ambientes debe confirmarse
con una exportación que preserve las columnas.

Consultar los endpoints internos y las cadenas de conexión en el inventario
autorizado de TGS. No utilizar este documento para deducir una conexión.

## Autenticación y credenciales

### Acceso a eTGS

Según la fuente, tanto la aplicación migrada como la anterior utilizan LDAP.
Para los ambientes bajos, de DEV a UAT, el documento indica que se utilizan
las mismas credenciales en ambas aplicaciones y en esos ambientes.
Esta es una descripción del origen, no una recomendación de reutilización.

Los usuarios tienen distintos permisos y roles. La fuente menciona que la
respuesta de login permite consultar esa información mediante Postman.
Cualquier comprobación debe hacerse únicamente con una cuenta y un ambiente
autorizados; no probar cuentas de terceros ni almacenar respuestas con tokens
en este repositorio.

### Gestión de secretos

Las credenciales, incluso las de pruebas, deben permanecer en el gestor de
secretos o bóveda autorizado por TGS/Baufest, con acceso controlado.
El Markdown puede documentar el procedimiento para solicitar acceso y una
referencia no secreta al registro correspondiente, pero no el valor del secreto.
No se proporcionó el gestor ni una referencia autorizada en esta importación.

Para automatizaciones, utilizar el almacén de credenciales aprobado de la
plataforma y su mecanismo de inyección en tiempo de ejecución. No incluir
secretos en código, documentación, logs, comandos registrados ni archivos
versionados de configuración.

Un archivo local excluido de Git no sustituye una bóveda. Su uso, si corresponde,
debe respetar el procedimiento aprobado del equipo y nunca exponerse en logs.
Si una credencial pudo quedar expuesta fuera de su canal autorizado, solicitar
al responsable su rotación o revocación. Borrarla de un documento no elimina
copias ni historial.

## Conexión con SAP

La aplicación migrada y la anterior tienen funcionalidades que consultan SAP.
Según el documento, pueden autenticarse con un usuario específico o con uno
por defecto. Para DESA se describía una cuenta por defecto compartida entre
ambas aplicaciones; sus valores no se importaron.

La fuente sitúa la configuración de la aplicación nueva en
`application.properties`. No se inspeccionó ese archivo ni se confirmó cómo se
inyectan actualmente las credenciales. Debe utilizarse el mecanismo autorizado
de configuración externa y secretos, sin escribir valores sensibles en Git.

## Infraestructura

La infraestructura se encuentra del lado del cliente y, en su mayoría, es
gestionada por él. Para algunas herramientas, el equipo puede solicitar
permisos y adaptaciones.

La fuente remite las URLs de herramientas a «TGS013 - Setup de Ambiente de
Trabajo». No se proporcionó el enlace de ese documento.

### CI/CD

Jenkins automatiza tareas relacionadas con la calidad de los repositorios,
la configuración, el despliegue y el mantenimiento de ambientes. La fuente
describe máquinas virtuales con sistema operativo Red Hat y servidor JBoss.

Esta descripción no autoriza ejecutar jobs ni modificar configuraciones.

### SCM

GitLab es el repositorio SCM indicado por la fuente. Los Merge Requests se
utilizan para la revisión de código entre pares y el aseguramiento de calidad
durante el ciclo de desarrollo.

El proceso se detalla en «TGS013 - Merge Requests», cuyo enlace no fue
proporcionado.

### Base de datos

La tecnología indicada es Oracle versión 19. Los endpoints, cuentas y secretos
deben consultarse por los canales autorizados.

## Información pendiente

- Tabla original con columnas preservadas para confirmar owners, managers,
  servidores y correspondencias de ambientes.
- Propósito y características de UPG y Vaca Muerta.
- Confirmación de si DESA y DEV son nombres equivalentes en este contexto.
- Vigencia de versiones, responsables e inventario de conexiones.
- Enlaces de «TGS013 - Setup de Ambiente de Trabajo» y «TGS013 - Merge Requests».
- Procedimiento de acceso al gestor de secretos autorizado, sin compartir
  contraseñas ni tokens en el chat o en el repositorio.
