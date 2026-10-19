---
name: fixy-implement-verify
description: "Usar despues del diagnostico de fixy para implementar una correccion completa, agregar regresion y verificar exactamente el comportamiento del bug en etgs-ui."
---

# Implementar y verificar

## Implementacion

1. Corrige la causa raiz con cambios precisos y completos en todas las
   superficies afectadas. Conserva convenciones, mensajes y comportamiento
   ajeno al bug. No refactorices ni actualices dependencias sin necesidad.
2. Agrega o ajusta una prueba de regresion con el runner existente. Demuestra
   el fallo anterior y el exito posterior cuando sea viable. No agregues un
   framework nuevo: ReactDOM y react-dom/test-utils ya forman parte del proyecto.
3. Verifica el criterio reportado, el caso valido relacionado y los bordes
   relevantes. Si hay un umbral, mide ese umbral; no pruebes una aproximacion.
4. Actualiza documentacion directamente relacionada cuando cambie un contrato
   o comportamiento documentado.

## Comandos y precauciones

Lee primero los scripts actuales de package.json. En PowerShell, para una prueba
enfocada no interactiva, reemplaza la ruta de ejemplo por el archivo real:

```powershell
$env:CI = 'true'
npm test -- --watch=false --runTestsByPath src\modulo\componente.test.jsx
```

`src\modulo\componente.test.jsx` es un ejemplo, no un archivo existente. No uses
`--passWithNoTests`: que no se ejecuten pruebas no significa que el bug paso.

- Ejecuta primero las pruebas enfocadas. Amplia el alcance segun las superficies
  compartidas modificadas o los fallos observados.
- El script `npm run lint` usa `--fix src` y puede cambiar archivos ajenos.
  Para revisar solo los archivos JS/JSX afectados sin autofix, usa el ESLint
  local (por ejemplo `.\node_modules\.bin\eslint.cmd <archivos-afectados>`),
  respetando la configuracion disponible. No inventes una configuracion para
  ocultar errores; informa si falta o esta bloqueada.
- Si la correccion requiere validar el bundle, usa `npm run build`. Este script
  genera CSS y metadatos de Git ademas del bundle: revisa el diff posterior y
  elimina solo artefactos propios, nunca cambios preexistentes del usuario.
- Si modificas SCSS, verifica tambien la compilacion con `npm run build-css`.
- Instala/restaura dependencias solo si cambiaste el manifiesto o un comando
  de validacion fallo por una dependencia ausente. No muestres configuracion
  privada ni credenciales para diagnosticar.
- Para validacion visual, usa herramientas de navegador si estan disponibles.
  Si levantas un servidor, confirma que responde, registra su identificador y
  detiene solo el proceso que creaste al terminar, salvo pedido contrario.

## Evaluacion

Registra comando, resultado y criterio cubierto. Distingue fallos nuevos de
fallos preexistentes con evidencia, no por suposicion. Un build exitoso no prueba
el comportamiento funcional del ticket.

Si no hay automatizacion viable, describe una validacion manual reproducible.
Si no puedes ejecutarla, marca el resultado como pendiente: no afirmes que
paso, no publiques una resolucion ni solicites el pase a QA.

Antes de publicar:

- Revisa el diff final y confirma que no incluye cambios ajenos ni secretos.
- Confirma que cada criterio de aceptacion tiene evidencia de validacion.
- Informa limitaciones reales. No hagas commit, push ni despliegues.

## Salida para Jira

Clave del ticket, causa raiz, resumen de cambios propios, criterios cubiertos,
comandos y resultados, pasos de revision/QA y limitaciones no bloqueantes.
Si un criterio esta pendiente o falla, el trabajo sigue incompleto y no pasa
a la skill de publicacion.
