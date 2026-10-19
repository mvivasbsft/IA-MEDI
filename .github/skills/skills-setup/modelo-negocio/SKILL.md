---
name: modelo-negocio
description: "Usar para explicar y validar el modelo de negocio de una funcionalidad TGS antes de tocar codigo: entidades, reglas, estados, restricciones, actores y excepciones."
argument-hint: "Requerimiento, ticket o pantalla cuyo modelo de negocio se debe analizar"
user-invocable: true
disable-model-invocation: true
---

# Modelo de negocio

## Objetivo y limites

Entender el comportamiento requerido antes de implementar. Esta skill es de
analisis: no modifica codigo, datos, tickets ni configuracion. Un modelo
propuesto no autoriza ejecutar una implementacion.

## Paso obligatorio: elegir modulo

Antes de analizar codigo o consultar tickets, pregunta siempre
"¿Sobre qué módulo desea realizar el análisis?" con `ask_user` y estas
opciones exactas: `COST`, `LIMA`, `MAC`, `MEDI`, `MOB`, `SPAC`.
Espera la seleccion; no elijas por el archivo abierto, un ticket o el contexto.
Si no hay herramienta de preguntas disponible, solicita la misma seleccion
y espera la respuesta antes de continuar.

Mapea la opcion a su carpeta en etgs-ui: COST -> `src/cost`,
LIMA -> `src/lima`, MAC -> `src/mac`, MEDI -> `src/medi`,
MOB -> `src/mob`, SPAC -> `src/spac`. Verifica que exista.
Si no existe o la respuesta no corresponde a una opcion, informa el problema
y solicita una seleccion valida; no uses otro modulo como fallback.

Limita el analisis al modulo seleccionado y, si hay un requerimiento puntual,
al flujo correspondiente dentro de el. Lee dependencias compartidas o externas
solo cuando sean necesarias para entender ese flujo y declara ese alcance.
No recorras todo el proyecto ni otros modulos de manera indiscriminada.
Si el requerimiento pertenece a otro modulo, aclara la discrepancia y pide
confirmar el modulo antes de continuar. Incluye modulo, carpeta y limites
efectivamente revisados en la salida.

## Procedimiento

1. Identifica el requerimiento, modulo y pantalla. Usa los datos aportados;
   pregunta solo por informacion necesaria que falte. Si una pantalla no
   puede identificarse con certeza, solicita su ruta o nombre completo.
2. Consulta la documentacion funcional disponible y el codigo del flujo
   relevante, incluyendo validaciones, servicios y pruebas. Si se aporta un
   ticket y hay una integracion autorizada, consultalo solo en lectura.
   Si no hay acceso, solicita su contenido sin simular una consulta exitosa.
3. Separa hechos documentados, comportamiento observado en codigo, hipotesis
   y propuestas. El codigo actual no prueba por si solo la regla deseada;
   ante contradicciones, presenta ambas fuentes y pide una decision.
4. Describe:
   - Entidades, identificadores, relaciones y ciclo de vida.
   - Actores, responsabilidades y permisos por operacion.
   - Reglas de negocio, formulas, unidades, precision y limites pertinentes.
   - Estados y transiciones: origen, evento, precondicion y destino.
   - Restricciones de integridad, fechas, cierres y dependencias relevantes.
   - Excepciones y resultado esperado ante errores, vacios o duplicados.
5. Para cada regla importante indica su fuente y un ejemplo verificable.
   No inventes formulas, roles, restricciones o estados no confirmados.
6. Convierte lo confirmado en criterios de aceptacion observables. Identifica
   preguntas bloqueantes antes de sugerir cambios; preguntalas una por vez
   usando la herramienta de preguntas cuando este disponible.

## Salida

- Resumen del objetivo y limites funcionales.
- Entidades y relaciones; actores y operaciones permitidas.
- Tabla de reglas con fuente, ejemplo y excepciones.
- Estados y transiciones relevantes.
- Criterios verificables y decisiones pendientes.

Enlaza las fuentes disponibles y marca expresamente lo no confirmado. No
presentes como aprobado un modelo que todavia contiene decisiones bloqueantes.
