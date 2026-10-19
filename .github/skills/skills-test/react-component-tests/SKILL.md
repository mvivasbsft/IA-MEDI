---
name: react-component-tests
description: "Usar al crear o depurar pruebas de componentes React en etgs-ui, verificando renderizado, eventos, estados y resultados visibles con las herramientas instaladas."
---

# Pruebas de componentes React

## Cuándo usar

- Probar el renderizado, las interacciones, los cambios de estado y las salidas visibles de componentes React.
- Investigar regresiones de UI en componentes, formularios o vistas.

## Procedimiento

1. Revisa el componente, sus dependencias y las pruebas existentes antes de elegir el enfoque. El proyecto usa React 16, `react-scripts test` y `jsdom`.
2. Usa primero las utilidades ya instaladas y los patrones del repositorio. `react-dom/test-utils` y `ReactDOM` forman parte de las dependencias de React; no asumas que React Testing Library o Enzyme están instalados.
3. Renderiza el componente en un nodo DOM de prueba. Usa `act` y utilidades como `Simulate` cuando corresponda; verifica texto, atributos, elementos y efectos observables en lugar de detalles internos de estado.
4. Cubre el estado inicial y las interacciones relevantes, incluyendo validación, estados vacíos o errores cuando formen parte del comportamiento esperado. Limpia el DOM y los mocks después de cada prueba, siguiendo los patrones locales.
5. Evita simular el comportamiento del componente bajo prueba. Si una dependencia requiere mock, limita el mock a esa frontera y comprueba lo que el usuario puede observar.
6. Ejecuta la prueba enfocada desde la raíz de `etgs-ui`:

   ```powershell
   $env:CI = 'true'
   npm test -- --watch=false --runTestsByPath src\modulo\componente.test.jsx
   ```

   Sustituye la ruta de ejemplo por el archivo real y no uses `--passWithNoTests`.
7. Si `jsdom` o un componente de UI no permite reproducir una interacción real del navegador, informa la limitación y recomienda una validación manual concreta; no afirmes que la prueba demuestra comportamiento visual real.

## Restricciones del proyecto

- No agregues dependencias de testing sin necesidad y aprobación de alcance.
- No uses snapshots extensos como único criterio para validar interacciones.
- Evita ejecutar `npm run lint` como verificación focal porque aplica autofix a `src`.
