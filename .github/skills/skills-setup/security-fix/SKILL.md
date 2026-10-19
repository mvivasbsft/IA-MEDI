---
name: security-fix
description: "Usar para analizar vulnerabilidades de una funcionalidad o dependencia y proponer parches minimos respetando arquitectura y comportamiento esperado; implementar solo con autorizacion explicita."
argument-hint: "Hallazgo, advisory o flujo vulnerable a analizar"
user-invocable: true
disable-model-invocation: true
---

# Analizar y proponer correcciones de seguridad

## Alcance y autorizacion

El comportamiento por defecto es analizar y proponer, sin editar archivos.
Aplicar un parche requiere una solicitud explicita del usuario.
Trabaja solo sobre sistemas y codigo autorizados; no pruebes explotaciones
contra produccion, no extraigas datos ni expongas credenciales.

## Paso obligatorio: elegir modulo

Antes de analizar codigo o delegar una revision de seguridad, pregunta siempre
"¿Sobre qué módulo desea realizar el análisis?" con `ask_user` y estas
opciones exactas: `COST`, `LIMA`, `MAC`, `MEDI`, `MOB`, `SPAC`.
Espera la seleccion; no elijas por el archivo abierto, un hallazgo o el contexto.
Si no hay herramienta de preguntas disponible, solicita la misma seleccion
y espera la respuesta antes de continuar.

Mapea la opcion a su carpeta en etgs-ui: COST -> `src/cost`,
LIMA -> `src/lima`, MAC -> `src/mac`, MEDI -> `src/medi`,
MOB -> `src/mob`, SPAC -> `src/spac`. Verifica que exista.
Si no existe o la respuesta no corresponde a una opcion, informa el problema
y solicita una seleccion valida; no uses otro modulo como fallback.

Limita el analisis, la delegacion y cualquier parche autorizado al flujo
afectado del modulo seleccionado. Incluye dependencias compartidas o externas
solo cuando sean necesarias para verificar la vulnerabilidad en ese modulo.
No recorras todo el proyecto ni otros modulos indiscriminadamente.
Si un parche compartido afecta otros modulos, explica ese impacto y solicita
autorizacion para ampliar el alcance antes de editar. Si el hallazgo no afecta
al modulo elegido, aclara la discrepancia y pide confirmar el modulo.
Incluye modulo, carpeta y limites efectivamente revisados en la salida.

## Procedimiento

1. Delimita el hallazgo, flujo y version afectada. Revisa las fuentes aportadas
   y las restricciones del entorno. Si hay un advisory, verifica producto,
   versiones vulnerables, condiciones y version corregida en una fuente
   confiable disponible. No asumas que una version antigua es explotable.
2. Para analizar vulnerabilidades, invoca primero el agente especializado
   security-review si esta disponible, con un alcance acotado y de solo lectura.
   No dupliques su analisis. Si no esta disponible, informa la limitacion y
   realiza una revision estatica acotada solo si el entorno lo permite.
   Respeta cualquier flujo de autorizacion y presentacion de hallazgos del
   entorno; esta skill no lo reemplaza.
3. Confirma la evidencia: entrada controlable, ruta alcanzable, validaciones
   existentes, operacion sensible, frontera de confianza y precondiciones.
   Distingue vulnerabilidad confirmada, hipotesis y falso positivo.
   No ejecutes payloads, scanners ni scripts de terceros para confirmarla.
4. Consulta la arquitectura y patrones del modulo. Propone una correccion
   en la frontera responsable sin alterar reglas funcionales ajenas.
   Nunca sustituyas autorizacion del backend por ocultar botones en React.
   Si la correccion pertenece a otro repositorio, identifica el contrato y
   el bloqueo en lugar de simular una solucion frontend.
5. Presenta para cada hallazgo:
   - Severidad, confianza, evidencia y precondiciones.
   - Archivos y frontera afectados; referencia OWASP/CWE cuando verificable.
   - Parche minimo propuesto y por que elimina la causa.
   - Compatibilidad, riesgos residuales y pruebas de regresion necesarias.
   - Si corresponde, mitigacion temporal claramente separada de la solucion.
6. Si el usuario solo pidio una propuesta, finaliza sin modificar codigo.
   Si pide aplicar el parche, confirma cualquier decision funcional ambigua
   antes de editar y conserva el estado previo del worktree.

## Implementacion autorizada

- Corrige la causa raiz siguiendo patrones locales, sin refactors ajenos.
- No desactives validaciones, permisos, TLS, CSP ni analisis de seguridad.
  No agregues catches silenciosos ni respuestas que simulen exito.
- Para dependencias, evita actualizaciones masivas. Verifica el advisory y
  la compatibilidad; usa el gestor y lockfile existentes. No ejecutes
  `npm audit fix --force` ni instales herramientas sin autorizacion.
- Agrega una regresion controlada que rechace el caso peligroso y preserve
  el valido. Usa pruebas locales aisladas sin llamadas reales a produccion.
- Ejecuta la validacion enfocada disponible; revisa el diff y reporta
  resultados reales y riesgos pendientes. Un scanner limpio no prueba por
  si solo que el defecto fue corregido.
- No hagas commit, push, despliegues ni publicaciones en Jira/PR sin una
  autorizacion independiente. No afirmes que un parche local ya esta desplegado.

## Salida

Resume alcance, hallazgos confirmados y descartados con evidencia, parches
propuestos o aplicados, comandos efectivamente ejecutados, resultados y
limitaciones. No declares una vulnerabilidad corregida si falta validar
la frontera responsable o un criterio requerido.
