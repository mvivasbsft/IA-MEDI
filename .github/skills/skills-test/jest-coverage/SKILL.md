---
name: jest-coverage
description: "Usar para medir, revisar o mejorar la cobertura de pruebas Jest en etgs-ui e identificar código no cubierto sin perseguir métricas artificiales."
---

# Cobertura de pruebas Jest

## Cuándo usar

- Generar un reporte de cobertura de pruebas.
- Investigar líneas o ramas sin cubrir y decidir qué pruebas de comportamiento faltan.
- Comparar la cobertura de un conjunto enfocado de pruebas antes y después de un cambio.

## Procedimiento

1. Lee el `package.json`, la configuración disponible y las pruebas existentes. El proyecto usa Jest a través de `react-scripts test --env=jsdom`.
2. Elige el alcance más pequeño que mida la lógica bajo análisis. Ejecuta desde la raíz de `etgs-ui`:

   ```powershell
   $env:CI = 'true'
   npm test -- --watch=false --coverage --runTestsByPath src\modulo\archivo.test.js
   ```

   Reemplaza la ruta de ejemplo por una prueba real. No uses `--passWithNoTests`.
3. Lee el resumen y los detalles de líneas, funciones y ramas. Identifica comportamientos importantes sin prueba y propón casos concretos, priorizando riesgos y criterios de aceptación.
4. Si agregas pruebas, comprueba que validen resultados observables y que fallen cuando el comportamiento correspondiente se rompe. Repite la medición con el mismo alcance para que la comparación sea significativa.
5. Reporta el comando, el alcance, los porcentajes disponibles y las áreas relevantes aún no cubiertas. Distingue la cobertura de la corrección funcional: un porcentaje alto por sí solo no demuestra que el comportamiento sea correcto.

## Precauciones

- No inventes ni impongas un umbral global si el repositorio no lo define.
- No agregues pruebas triviales ni modifiques código de producción solo para inflar porcentajes.
- No borres ni sobrescribas reportes preexistentes sin confirmar que fueron generados por esta ejecución.
- Si el runner no genera un reporte, informa el error real; no presentes un cálculo estimado como resultado.
