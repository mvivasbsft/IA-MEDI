---
name: jest-unit-tests
description: "Usar al crear, ejecutar o depurar pruebas unitarias con Jest en etgs-ui, especialmente para funciones y lógica de negocio JavaScript."
---

# Pruebas unitarias con Jest

## Cuándo usar

- Crear o modificar pruebas unitarias de funciones, validaciones, transformaciones y lógica de negocio.
- Ejecutar pruebas Jest enfocadas o diagnosticar por qué fallan.

## Procedimiento

1. Lee el `package.json` y la implementación actual del módulo. Este proyecto ejecuta Jest mediante `react-scripts test --env=jsdom`; no asumas una configuración independiente de Jest.
2. Busca pruebas cercanas y sigue su ubicación, nombres, estilo y convenciones. Usa el sufijo de prueba que ya tenga el proyecto; si no hay precedentes, usa `.test.js` o `.test.jsx` junto al módulo.
3. Prueba comportamiento observable: entradas válidas, límites relevantes, entradas inválidas y errores cuando apliquen. Mantén cada prueba enfocada y evita duplicar la implementación dentro de la expectativa.
4. Usa los mocks solo cuando sean necesarios para aislar una dependencia externa. Limpia los mocks entre pruebas y no simules el resultado que la prueba necesita demostrar.
5. Ejecuta primero la prueba afectada desde la raíz de `etgs-ui`:

   ```powershell
   $env:CI = 'true'
   npm test -- --watch=false --runTestsByPath src\modulo\archivo.test.js
   ```

   Reemplaza la ruta de ejemplo por la prueba real. No uses `--passWithNoTests`.
6. Informa el comando ejecutado, el resultado y cualquier limitación. No afirmes que una prueba pasó si no llegó a ejecutarse.

## Restricciones del proyecto

- No agregues un runner, framework o dependencia de pruebas si las herramientas existentes bastan.
- Evita `npm run lint` como verificación focal: su script usa `eslint --fix src` y puede modificar archivos ajenos.
- No cambies código de producción cuando la solicitud sea únicamente crear o ejecutar pruebas.
