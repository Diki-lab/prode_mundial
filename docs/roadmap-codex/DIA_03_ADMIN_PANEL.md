# Dia 03 - Admin Panel

## Estado

COMPLETADO ✅

Tests locales: OK ✅

Validacion en navegador: OK ✅

## Objetivo

Convertir la seccion admin actual en un panel claro para operar el torneo: bloqueos, resultados, jugadores y estado del sistema.

## Dependencia estricta

No puede comenzar si `DIA_02_BLOQUEOS.md` no esta marcado como `COMPLETADO`.

## Tareas

- Verificar que Dia 2 este `COMPLETADO`.
- Separar visualmente funciones admin: resultados, bloqueos, jugadores y diagnostico.
- Ocultar accesos admin para usuarios comunes.
- Mantener guards `isAdmin` en cada mutacion.
- Agregar confirmaciones para borrar jugadores o cambios destructivos.
- Evitar que admin borre o pise pronosticos de otros usuarios desde una implementacion parcial.
- Mostrar email/rol admin actual.

## Archivos permitidos

- `index.html`.
- `docs/AVANCE_PROYECTO.md`.
- `docs/FUNCIONALIDADES_PENDIENTES.md`.
- `docs/roadmap-codex/DIA_03_ADMIN_PANEL.md`.

## Archivos prohibidos

- `index_backup.html`.
- `index_backup_20260527_1651.html`.
- `prode_mundial_2026_web.html`.
- Imagenes.
- Cambios estructurales que requieran build.

## Criterios de aceptacion

- Usuario comun no ve panel admin ni accesos admin.
- Usuario comun no puede ejecutar funciones admin desde la UI.
- Admin ve controles de bloqueo, resultados y jugadores.
- Admin puede cargar resultados reales.
- Admin puede agregar jugador sin duplicados obvios por espacios/case.
- Acciones destructivas piden confirmacion.

## Tests obligatorios

- Login usuario comun y verificar ausencia de admin.
- Login admin y verificar panel.
- Cargar resultado como admin.
- Agregar jugador como admin.
- Intentar duplicado simple de jugador.
- Probar confirmacion de borrado.

## Resultado esperado

La operacion diaria del Prode queda concentrada en un panel admin entendible y menos riesgoso.

## Resultado de ejecucion

- Se oculto el acceso Admin del header para usuarios comunes.
- `showTab('jugadores')` ahora bloquea acceso si el usuario no es admin.
- Se agrego estado de identidad admin con email y rol.
- Se agrego acceso rapido desde Admin para cargar resultados en Fixture.
- Se mantuvieron controles separados para bloqueos y jugadores.
- Se agrego confirmacion antes de borrar jugadores.
- Se agrego validacion basica contra duplicados de jugadores por espacios y mayusculas/minusculas.
- La funcion `deletePrediction()` ya no borra pronosticos desde la vista admin parcial; informa que los pronosticos consolidados son solo lectura.

## Funciones nuevas

- `normalizePlayerName()`
- `playerExists()`

## Funciones modificadas

- `showTab()`
- `renderLockStatuses()`
- `updateAdminUI()`
- `deletePrediction()`
- `deletePlayer()`
- `addPlayer()`
- `runTests()`

## Tests ejecutados

- Sintaxis de scripts embebidos en `index.html`: OK.
- Test logico de normalizacion de jugador: OK.
- Test logico de deteccion de duplicado simple: OK.
- Usuario comun no ve acceso Admin.
- Usuario comun no puede entrar a jugadores/admin.
- Admin ve panel admin.
- Admin ve email/rol.
- Admin tiene acceso a carga de resultados.
- Agregar jugador duplicado muestra aviso.
- Borrar jugador pide confirmacion.
- Pronosticos consolidados quedan solo lectura.

## Tests manuales obligatorios para navegador

- Login usuario comun y verificar ausencia de Admin en header, tabs y cards.
- Intentar abrir panel admin como usuario comun y confirmar bloqueo.
- Login admin y verificar panel.
- Verificar que se muestra email/rol admin.
- Ir desde Admin a carga de resultados.
- Agregar jugador como admin.
- Intentar duplicado simple de jugador.
- Probar confirmacion de borrado.
- Verificar que la vista de pronosticos admin sigue en solo lectura.

## Prompt para iniciar el dia

```text
Ejecuta el Dia 3 - Admin Panel. Primero verifica que DIA_02_BLOQUEOS.md este COMPLETADO. Si no lo esta, detente. Si esta completo, mejora solo el panel admin y sus guards, manteniendo la app estatica.
```
