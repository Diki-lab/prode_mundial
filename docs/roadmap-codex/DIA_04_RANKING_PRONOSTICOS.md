# Dia 04 - Ranking Y Pronosticos

## Estado

COMPLETADO ✅

Tests locales: OK ✅

Validacion en navegador con varios usuarios: pendiente de ejecucion manual.

## Objetivo

Dejar confiables los pronosticos por usuario y el ranking, evitando duplicaciones entre datos legacy y subcoleccion Firestore.

## Dependencia estricta

No puede comenzar si `DIA_03_ADMIN_PANEL.md` no esta marcado como `COMPLETADO`.

## Tareas

- Verificar que Dia 3 este `COMPLETADO`.
- Revisar carga de `prode/mundial2026/predictions/{uid}`.
- Revisar fallback legacy de `prode/mundial2026.predictions`.
- Evitar doble conteo si existe el mismo pronostico en legacy y subcoleccion.
- Mostrar progreso de pronosticos cargados por usuario.
- Mejorar mensajes al guardar pronosticos.
- Validar inputs: numeros, negativos, campos vacios y partido seleccionado.
- Mantener regla de puntos: exacto 3, ganador/empate correcto 1.

## Archivos permitidos

- `index.html`.
- `docs/AVANCE_PROYECTO.md`.
- `docs/FUNCIONALIDADES_PENDIENTES.md`.
- `docs/roadmap-codex/DIA_04_RANKING_PRONOSTICOS.md`.

## Archivos prohibidos

- `index_backup.html`.
- `index_backup_20260527_1651.html`.
- `prode_mundial_2026_web.html`.
- Reglas Firebase salvo necesidad detectada en tests.
- Imagenes.

## Criterios de aceptacion

- Cada usuario ve y edita solo sus pronosticos.
- Ranking usa predicciones consolidadas sin duplicar.
- Podio de inicio coincide con ranking completo.
- Los puntos calculan correctamente.
- Guardar pronostico muestra feedback claro.
- Inputs invalidos no guardan datos corruptos.

## Tests obligatorios

- Usuario A guarda pronostico.
- Usuario B guarda otro pronostico.
- Ranking muestra ambos usuarios.
- Resultado exacto suma 3.
- Ganador/empate correcto suma 1.
- Resultado incorrecto suma 0.
- Podio coincide con top 3 del ranking.

## Resultado esperado

Pronosticos y ranking quedan listos para uso real por varios participantes.

## Resultado de ejecucion

- Se agrego deduplicacion de pronosticos por jugador y partido.
- El ranking usa como fuente principal la subcoleccion `prode/mundial2026/predictions`.
- El array legacy `prode/mundial2026.predictions` queda solo como fallback si no hay documentos en la subcoleccion.
- Se mantiene fallback local si no hay datos en subcoleccion ni legacy.
- Se mejoro la validacion de inputs para evitar guardar campos vacios como cero.
- Se agrego progreso visible en la pantalla de pronosticos.
- El perfil muestra `Mis pronosticos` como progreso `cargados/total`.
- La vista admin muestra cantidad consolidada de pronosticos unicos.

## Funciones nuevas

- `getPredictionKey()`
- `dedupePredictionList()`
- `getPredictionDocsList()`
- `getRankingPredictionSource()`
- `getPredictionProgress()`

## Funciones modificadas

- `updatePredictionUserStatus()`
- `applyProfile()`
- `loadCloudData()`
- `saveCurrentUserPredictions()`
- `savePrediction()`
- `renderPredictions()`
- `getRankingTotals()`
- `runTests()`

## Tests ejecutados

- Sintaxis de scripts embebidos en `index.html`: OK.
- Test logico de deduplicacion por jugador y partido: OK.
- Test logico de prioridad de subcoleccion sobre legacy: OK.
- Test logico de progreso contra total de partidos: OK.
- Tests de puntaje existentes se mantienen: exacto 3, ganador/empate 1, incorrecto 0.

## Tests manuales obligatorios para navegador

- Usuario A guarda pronostico y se confirma escritura en `prode/mundial2026/predictions/{uid}`.
- Usuario B guarda pronostico y se confirma escritura en su propio `{uid}`.
- Ranking muestra ambos usuarios.
- Verificar que no se duplican puntos si existen datos legacy.
- Verificar que `Mis pronosticos` muestra progreso correcto.
- Verificar que el podio coincide con el top 3 del ranking.
- Intentar guardar inputs vacios o negativos y confirmar mensaje de error.

## Prompt para iniciar el dia

```text
Ejecuta el Dia 4 - Ranking y Pronosticos. Primero verifica que DIA_03_ADMIN_PANEL.md este COMPLETADO. Si no lo esta, detente. Si esta completo, enfocate en consistencia de pronosticos por usuario y ranking sin duplicados.
```
