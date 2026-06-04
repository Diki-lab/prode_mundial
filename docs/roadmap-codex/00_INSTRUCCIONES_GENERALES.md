# Roadmap Codex - Instrucciones Generales

## Estado

PENDIENTE.

Este documento define como ejecutar el roadmap funcional del Prode Mundial 2026. La app principal vive en `index.html`, usa HTML/CSS/JS vanilla, Firebase por CDN, `localStorage` como respaldo y GitHub Pages como despliegue esperado.

## Objetivo

Terminar la parte funcional minima para publicar el Prode antes del Mundial, priorizando seguridad, bloqueos, pronosticos por usuario, ranking confiable, panel admin y validacion completa del flujo.

## Tareas

- Trabajar los dias en orden estricto.
- Antes de iniciar cada dia, verificar que el dia anterior este marcado como `COMPLETADO`.
- No avanzar a una fase posterior si la anterior dejo pruebas fallidas, reglas sin publicar o comportamiento ambiguo.
- Mantener la app como estatica y compatible con GitHub Pages.
- Evitar migraciones grandes, frameworks, backend nuevo o build step durante este roadmap.
- Registrar cualquier decision funcional relevante en `docs/AVANCE_PROYECTO.md` o `docs/FUNCIONALIDADES_PENDIENTES.md`.

## Dependencia estricta

- Dia 1 puede comenzar cuando este aprobado este roadmap.
- Dia 2 no puede comenzar si Dia 1 no esta `COMPLETADO`.
- Dia 3 no puede comenzar si Dia 2 no esta `COMPLETADO`.
- Dia 4 no puede comenzar si Dia 3 no esta `COMPLETADO`.
- Dia 5 no puede comenzar si Dia 4 no esta `COMPLETADO`.
- Dia 6 no puede comenzar si Dia 5 no esta `COMPLETADO`.
- Dia 7 no puede comenzar si Dia 6 no esta `COMPLETADO`.

## Archivos permitidos

- `index.html`, solo cuando el dia correspondiente lo permita.
- `docs/AVANCE_PROYECTO.md`.
- `docs/MEMORIA_PROYECTO.md`.
- `docs/FUNCIONALIDADES_PENDIENTES.md`.
- `docs/roadmap-codex/*.md`.
- Archivos nuevos de reglas Firebase si se acuerda versionarlas: `firestore.rules`, `storage.rules`.

## Archivos prohibidos

- `index_backup.html`.
- `index_backup_20260527_1651.html`.
- `prode_mundial_2026_web.html`.
- `.git/`.
- Imagenes salvo pedido explicito.
- Cualquier archivo fuera del workspace.

## Criterios de aceptacion

- Cada dia termina con estado `COMPLETADO` solo si sus tests obligatorios pasan o si queda documentado por que una prueba externa no pudo ejecutarse.
- No se introducen dependencias que requieran build.
- No se borra ni recrea `prode/mundial2026`.
- No se vuelve a login hardcodeado.
- El admin oficial sigue siendo `dmcfarlane@prode.local`.

## Tests obligatorios

- Revisar consola del navegador y confirmar que no haya errores nuevos.
- Probar login/logout cuando el dia toque auth o datos.
- Probar flujo jugador cuando el dia toque pronosticos.
- Probar flujo admin cuando el dia toque resultados, bloqueos o panel.
- Probar visualmente desktop y mobile antes de publicar.

## Resultado esperado

Una secuencia de trabajo clara, ejecutable en 7 dias, con el menor riesgo posible para publicar una version funcional del Prode.

## Prompt para iniciar el dia

Usar el archivo del dia correspondiente y decir:

```text
Ejecuta el Dia N del roadmap Codex. Antes de tocar codigo, verifica que el dia anterior este COMPLETADO. Si la dependencia no esta cumplida, detente y explica que falta.
```
