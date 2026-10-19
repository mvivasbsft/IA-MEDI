---
name: test-regression-bug
description: "Usar para convertir un bug reproducible o un criterio de aceptación en una prueba de regresión enfocada con Jest para etgs-ui."
---

# Pruebas de regresión para bugs

## Cuándo usar

- Añadir una prueba que proteja contra la reaparición de un bug confirmado.
- Traducir pasos de reproducción y resultados esperados a aserciones automatizadas.
- Evaluar si una prueba existente detectaría una regresión concreta.

## Procedimiento

1. Obtén pasos de reproducción y el resultado esperado. Distingue los hechos confirmados de las hipótesis; si falta información para definir una aserción, solicita una aclaración.
2. Sigue el flujo afectado desde la UI o función hasta la lógica responsable. Revisa `package.json` y las pruebas cercanas antes de elegir el archivo y el tipo de prueba.
3. Escribe la prueba más pequeña que reproduzca el comportamiento reportado y compruebe el resultado esperado. Incluye el caso válido relacionado si es necesario para demostrar que el arreglo no rompe el flujo normal.
4. Cuando sea seguro y factible, ejecuta primero la prueba contra el comportamiento defectuoso y confirma que falla por la aserción esperada; luego ejecútala contra la corrección y confirma que pasa. No reviertas cambios del usuario ni modifiques producción solo para fabricar una falla.
5. Si el bug ya está corregido y no puedes ejecutar la prueba contra la versión defectuosa, explica esa limitación y valida que la prueba distingue el resultado incorrecto descrito del correcto.
6. Ejecuta la prueba focalizada con el runner existente, desde la raíz de `etgs-ui`:

   ```powershell
   $env:CI = 'true'
   npm test -- --watch=false --runTestsByPath src\modulo\archivo.test.js
   ```

   Reemplaza la ruta de ejemplo por la prueba real. No uses `--passWithNoTests`.
7. Informa qué comportamiento cubre, si se observó la falla previa, el resultado posterior y cualquier limitación. No confundas éxito de compilación con reproducción o corrección del bug.

## Límites

- Esta skill se enfoca en diseñar y validar la prueba. Para diagnosticar una causa raíz de un ticket Jira o implementar una corrección, sigue el flujo de Fixy.
- No inventes criterios de aceptación ni realices operaciones contra producción.
- No agregues un framework de pruebas ni cambies configuración global de Jest para un caso puntual.
