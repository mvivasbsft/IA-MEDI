---
name: fixy-jira-publish
description: "Usar solo tras corregir y validar el bug con fixy para comentar automaticamente en Jira y pasar a una transicion real de revision/QA sin cerrar el ticket."
---

# Publicar el resultado y pasar a revision/QA

## Precondiciones

- El ticket esta identificado sin ambiguedad.
- La correccion esta implementada y cada criterio requerido fue validado.
- No hay fallos ni decisiones pendientes que bloqueen el resultado.
- El usuario autorizo este flujo de comentario y pase a revision/QA.

Esta skill no autoriza commit, push, PR, cierre del ticket ni despliegue.

## Procedimiento

1. Usa directamente el MCP `atlassian` de `.vscode/mcp.json`, sin GitKraken ni
   agentes intermediarios, con el sitio y ticket confirmados en la etapa de
   contexto. Descubre las operaciones y esquemas disponibles (mediante
   descubrimiento y ejecucion si asi los expone el servidor).
   Consulta el estado actual, los comentarios recientes y las transiciones
   disponibles con sus identificadores, destinos y campos obligatorios.
   Consulta datos frescos: no adivines nombres ni IDs. Si falta una operacion
   necesaria o su permiso, informa el bloqueo; no uses otra integracion.
2. Si el ticket se cerro/resolvio o su contexto cambio de forma incompatible
   durante el trabajo, detente y consulta al usuario.
3. Prepara un comentario breve con el formato de abajo. Aclara que la correccion
   esta en el worktree local, sin commit, push ni despliegue; QA necesita que
   esos cambios sean integrados por el flujo del equipo. No publiques codigo
   completo, secretos, rutas locales privadas ni adjuntos sensibles.
4. Evita duplicar un comentario del mismo intento si ya fue publicado. Usa la
   operacion MCP de agregar comentario y conserva el ID o URL devuelto.
5. Selecciona solo una transicion disponible cuyo destino sea revision/QA
   (por ejemplo Code Review, Ready for QA o En revision). Son ejemplos, no
   nombres garantizados. Si hay varios destinos razonables, pregunta cual
   corresponde; si falta un campo obligatorio, pide su valor. Nunca inventes
   el estado, una asignacion ni una resolucion.
6. Si ya esta en el destino apropiado, no repitas la transicion. Si no existe
   una transicion de revision/QA disponible, deja el comentario y comunica el
   bloqueo; no lo reemplaces por Resuelto o Cerrado.
7. Ejecuta mediante el MCP la transicion seleccionada solo despues de confirmar
   la publicacion del comentario. Respeta permisos y aprobaciones.
8. Verifica mediante una nueva lectura que el comentario existe y el estado es
   el esperado. Reporta el resultado real de cada operacion por separado.

Si hay un timeout o resultado ambiguo de escritura, consulta primero el ticket
antes de reintentar para no duplicar comentarios ni transiciones. Si el servicio
no permite verificar, informa "no confirmado"; no conviertas el error en exito.

## Plantilla del comentario

```text
Fixy - correccion implementada localmente para <CLAVE>

Causa raiz: <evidencia breve>
Cambios: <resumen y rutas relativas de archivos propios>
Validacion: <criterios verificados, comandos y resultados reales>
Revision/QA: <pasos concretos para revisar la correccion>
Limitaciones: <limitaciones no bloqueantes, si existen>

Cambios en worktree local. Sin commit, push ni despliegue.
La validacion en QA requiere integrar estos cambios mediante el flujo del equipo.
```

## Resultado

Devuelve enlace del ticket, ID/URL del comentario y estado confirmado.
Si el comentario se publico pero la transicion fallo, conserva ese resultado
parcial y explica el bloqueo sin duplicar la publicacion.
