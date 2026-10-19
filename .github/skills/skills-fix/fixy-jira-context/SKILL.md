---
name: fixy-jira-context
description: "Usar al iniciar fixy con una clave o URL de Jira para obtener el bug, su comportamiento esperado y criterios verificables antes de tocar codigo."
---

# Obtener contexto del bug

## Procedimiento

1. Extrae una sola clave Jira de la entrada, por ejemplo `ARTGS013-17157` de
   `https://baufest.atlassian.net/browse/ARTGS013-17157`. Valida el formato de
   proyecto y numero (`^[A-Z][A-Z0-9_]*-[0-9]+$`). Si hay varios tickets o una
   entrada ambigua, pregunta cual resolver. No sigas enlaces a otros hosts sin
   verificar que pertenecen a la instancia Jira autorizada.
2. Usa directamente el MCP `atlassian` de `.vscode/mcp.json`, sin GitKraken ni
   agentes intermediarios. Descubre las operaciones Jira y sus esquemas
   disponibles; si el servidor expone descubrimiento y ejecucion, usa ese flujo.
   Identifica el sitio autorizado a partir de la URL del ticket o de los sitios
   accesibles al usuario. Para una clave sin URL, usa `baufest.atlassian.net`;
   si no esta accesible, informa el bloqueo sin consultar otro sitio.
   Obten el identificador del sitio desde el MCP, sin inventarlo.
   Ejecuta una consulta de solo lectura de esa clave:
   resumen, tipo, descripcion, estado, pasos para reproducir, resultado actual
   y esperado, criterios de aceptacion, entorno/version y comentarios relevantes.
   Incluye contexto de adjuntos y tickets vinculados solo si es pertinente y
   accesible. No solicites descargar o ejecutar archivos desconocidos.
3. Comprueba que se trata del bug indicado. Si ya esta cerrado/resuelto, consulta
   al usuario antes de implementar o intentar cambiar su estado.
4. Convierte la informacion en criterios observables. Distingue hechos del
   ticket, evidencia del codigo e hipotesis. No agregues requisitos inventados.
5. Si faltan datos esenciales para distinguir un bug de un comportamiento
   esperado, pregunta de forma concreta. Si Jira no es accesible, informa el
   error y pide descripcion, reproduccion y resultado esperado; no simules una
   consulta exitosa. Con datos aportados por el usuario se puede continuar
   localmente, pero la publicacion en Jira queda bloqueada hasta recuperar acceso.

## Salida para la siguiente etapa

- Clave, resumen y estado actual confirmados, o limitacion de acceso.
- Pasos de reproduccion y comportamiento actual/esperado.
- Criterios de aceptacion verificables y contexto tecnico relevante.
- Datos faltantes que bloquean el diagnostico, si los hay.

## Ejemplo de consulta mediante MCP Atlassian

> Usa el MCP atlassian para consultar en modo solo lectura ARTGS013-17157
> en baufest.atlassian.net.
> Devuelve descripcion, estado, reproduccion, resultado esperado, criterios,
> entorno y comentarios relevantes. No modifiques el ticket. Informa si algun
> dato o permiso no esta disponible.

No afirmes que ese ticket tiene un problema concreto hasta recibir sus datos.
