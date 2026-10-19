---
name: fixy-diagnose
description: "Usar con el contexto de un bug Jira confirmado para rastrear su causa raiz en etgs-ui, reproducirlo y delimitar una correccion sin cambios ajenos."
---

# Diagnosticar la causa raiz

## Contexto del repositorio

- Es una UI React 16 con JavaScript/JSX, Material UI 4 y react-scripts 2.
- Los modulos funcionales estan bajo `src`; hay utilidades, llamadas y manejo
  de mensajes compartidos en `src/common`.
- Consulta siempre el package.json actual: esta skill no sustituye la
  configuracion ni las convenciones de los archivos involucrados.

## Procedimiento

1. Lee el estado y diff del worktree sin modificarlo para identificar cambios
   previos. No reviertas ni atribuyas cambios del usuario a fixy.
2. Busca la pantalla, texto, ruta, componente o servicio mencionado en el ticket.
   Lee el codigo relacionado antes de editar. Usa busquedas acotadas y sigue el
   flujo real entre UI, estado, validacion, servicio y respuesta.
3. Busca soluciones similares y helpers existentes. Reutiliza los patrones
   locales, incluidos manejo de errores, permisos, formatos y mensajes.
4. Reproduce el fallo con una prueba enfocada o una interaccion controlada en la
   UI cuando sea posible. No pruebes operaciones con efectos reales en produccion.
   Si no puedes reproducirlo, distingue evidencia estatica de reproduccion real.
5. Explica la causa raiz con evidencia concreta y conecta cada criterio de
   aceptacion con las superficies que deben cambiar. Considera estados vacios,
   errores, permisos y limites relevantes al bug, no casos ajenos.
6. Si el problema pertenece al backend u otro repositorio, informa el contrato
   observado y el bloqueo. No ocultes un error de backend con un fallback de UI
   que simule exito. Pregunta si hace falta acceso o una decision funcional.

## Salida para la implementacion

- Causa raiz y evidencia, o hipotesis aun pendientes.
- Archivos y superficies afectados.
- Correccion propuesta y comportamientos existentes que deben preservarse.
- Prueba de regresion y validaciones necesarias para cada criterio.

## Ejemplo

Si el ticket dice que un campo obligatorio permite guardar vacio, rastrea el
formulario, la validacion y la llamada de guardado. No agregues solo un asterisco
al label: demuestra que el valor invalido no dispara el guardado y que un valor
valido sigue funcionando.
