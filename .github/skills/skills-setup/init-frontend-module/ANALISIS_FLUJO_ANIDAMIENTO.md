# Estado del flujo de anidamiento

Resumen del comportamiento vigente de `init-frontend-module`, contrastado con
`SKILL.md` y `scripts/create-tgs-module.ps1` al 2026-10-05. El caso que motivo
este analisis se conserva como antecedente; las validaciones descritas abajo ya
estan implementadas.

## 1. Flujo actual

### 1.1 Disparo

El usuario pide crear un módulo frontend SPAC o MEDI, con o sin parámetros
en el mensaje.

### 1.2 Paso 1 — Recopilar parámetros (`SKILL.md`)

Si faltan parámetros, el Skill los recopila en este orden:

1. `ModuleName`
2. `Category` (SPAC/MEDI)
3. `ProjectPath`
4. Ubicacion simple o anidada; despues resuelve `FolderName` con el filesystem.

Si el usuario ya pasó los parámetros, no se pregunta nada.

En ubicacion simple, el Skill lista las carpetas directas y permite elegir una
o crear una nueva. En ubicacion anidada, permite navegar subcarpeta por
subcarpeta, crear niveles, usar la ruta actual como destino, ingresar una ruta
manual o cancelar.

### 1.3 Paso 2 — Ejecutar (`scripts/create-tgs-module.ps1`)

```powershell
create-tgs-module.ps1 -ProjectPath <path> -Category <SPAC|MEDI> -FolderName <folder> -ModuleName <name>
```

El script:

1. Valida que `ProjectPath` exista (si no, aborta).
2. Normaliza `Category` a mayúsculas.
3. Aborta si `ProjectPath/FolderName/ModuleName` ya existe, tanto si tiene
  contenido como si esta vacio. No existe el parametro `-Force`.
4. Crea `ProjectPath/FolderName` con `New-Item -Force`, que crea los niveles
  intermedios aunque no existan, y luego crea la estructura y los tres
  archivos base.
5. Si se usa `-Integrate`, valida que haya al menos un rol existente, modifica
  el menu y `Module.jsx`, y verifica que `roles.js` no haya cambiado.

### 1.4 Antecedente y proteccion vigente

El orden `Category → ModuleName → FolderName` no permite distinguir estos
dos casos, que usan el mismo nombre ("Balance") con significados distintos:

- **(a) Módulo nuevo, standalone**: "Balance" es el nombre de la nueva
  pantalla, sin relación con nada existente.
- **(b) Módulo dentro de jerarquía existente**: "Balance" ya es un módulo
  padre real (`src/medi/Balance`) que ya tiene un submódulo `Configuracion`,
  y lo que se quiere es agregar una pantalla nueva **dentro** de esa
  jerarquía (`Balance/Configuracion/<NuevoSubmodulo>`).

Ese caso motivo la incorporacion de la navegacion guiada y la validacion de
colision. Actualmente, el Skill comprueba el destino antes de ejecutar y el
script vuelve a comprobarlo como proteccion. Si existe, la creacion se aborta;
no pide confirmacion para sobrescribir.

## 2. Comportamiento actual y limites

La integracion es opcional y requiere roles existentes. El script agrega una
clave de menu solo si no existe; si ya existe, conserva su valor, aunque no
coincida con el `MenuLabel` solicitado. Agrega roles faltantes al import de
roles y reemplaza el import del componente para ubicarlo en el ancla prevista.
Si la ruta ya existe para el mismo componente, conserva el `RouteProtected`
sin reconciliar roles u otros valores; si pertenece a otro componente, aborta.

Por lo tanto, algunas comprobaciones de integracion toleran registros
existentes, pero la ejecucion completa no es idempotente: la validacion del
directorio impide repetir el generador para un modulo ya creado. Tampoco hay
rollback: una falla despues de crear el modulo o escribir el menu puede dejar
cambios parciales que deben reportarse y revisarse manualmente.

El script no verifica que las carpetas de `FolderName` existan antes de crear;
las crea. La navegacion y validacion de rutas manuales pertenecen al Skill.

## 3. Estado de los cambios

| Area | Estado actual |
|---|---|
| Seleccion guiada simple/anidada | Implementada en `SKILL.md` |
| Creacion de niveles intermedios | Implementada en el script |
| Colision de modulo existente | Se aborta; no hay sobreescritura ni `-Force` |
| Validacion de roles | Implementada durante la integracion; `roles.js` no se modifica |
| Menu, import y ruta | Integracion implementada con los limites descritos arriba |
| Confirmacion/rollback transaccional | El script no pregunta ni revierte etapas anteriores |
