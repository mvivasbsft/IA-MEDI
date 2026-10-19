---
name: modelo-arquitectura
description: "Usar para validar capas, dependencias, contratos, naming, integracion y patrones de arquitectura Baufest - TGS contra documentacion y evidencia del repositorio."
argument-hint: "Modulo, propuesta de cambio o flujo cuya arquitectura se debe validar"
user-invocable: true
disable-model-invocation: true
---

# Modelo de arquitectura

## Objetivo y limites

Evaluar una propuesta o implementacion contra la arquitectura verificable del
equipo. Solo analizar: no generar modulos, refactorizar ni cambiar contratos.
No atribuir una preferencia personal a un estandar de Baufest - TGS.

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

Limita el analisis al modulo seleccionado y, si hay un cambio puntual,
al flujo correspondiente dentro de el. Lee dependencias compartidas o externas
solo cuando sean necesarias para entender ese flujo y declara ese alcance.
No recorras todo el proyecto ni otros modulos de manera indiscriminada.
Si la propuesta pertenece a otro modulo, aclara la discrepancia y pide
confirmar el modulo antes de continuar. Incluye modulo, carpeta y limites
efectivamente revisados en la salida.

## Procedimiento

1. Delimita el modulo, cambio y repositorios disponibles. Lee sus instrucciones,
   documentacion de arquitectura, decisiones registradas y package.json.
   Distingue IA-MEDI, que contiene personalizaciones, del codigo de etgs-ui.
2. Busca patrones cercanos y componentes compartidos existentes. Diferencia
   normas explicitas de convenciones observadas; una coincidencia aislada
   no constituye una regla del equipo.
3. Traza las responsabilidades y dependencias del flujo: pantalla, componentes,
   contexto/estado, hooks, servicios, utilidades y sistemas externos. Usa la
   estructura real del modulo; no impongas capas inexistentes.
4. Revisa segun corresponda:
   - Separacion de UI, estado, reglas de negocio y acceso a datos.
   - Direccion de dependencias, ciclos, reutilizacion y duplicacion.
   - Contratos: entradas, salidas, tipos, errores, asincronia y compatibilidad.
   - Naming, ubicacion de archivos y convenciones locales de imports.
   - Integracion de menu, rutas, permisos y registro de actividad.
   - Manejo de errores, carga, vacios y limpieza de efectos.
   - Compatibilidad con las versiones instaladas y estrategia de pruebas.
5. Para MEDI, consulta las referencias pertinentes de
   [crear-pantalla](../crear-pantalla/SKILL.md) o
   [init-frontend-module](../init-frontend-module/SKILL.md) cuando apliquen.
   Leerlas no autoriza ejecutar sus scripts ni inicializar el modulo.
6. Clasifica cada punto como conforme, desviacion comprobada o no verificable.
   Adjunta la norma o evidencia y el impacto. Si las fuentes se contradicen
   o faltan contratos del backend, explicita la duda y pide confirmacion.
7. Propone ajustes minimos compatibles con el comportamiento esperado.
   No sugieras nuevas dependencias, patrones o migraciones sin necesidad.

## Salida

- Alcance, fuentes y mapa breve de responsabilidades/dependencias.
- Tabla: aspecto, norma o patron observado, evidencia, estado e impacto.
- Recomendaciones priorizadas y validacion requerida.
- Contratos, decisiones o accesos pendientes.

No declares conformidad con estandares privados no disponibles. No hagas
commit, push, cambios en Jira ni operaciones contra produccion.
