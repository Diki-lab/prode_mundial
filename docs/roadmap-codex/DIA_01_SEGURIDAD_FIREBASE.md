# Dia 01 - Seguridad Firebase

## Estado

COMPLETADO ✅

Tests: OK ✅

Validado por el usuario: `firestore.rules` fue creado, compilado correctamente, desplegado correctamente en Firebase, `firebase.json` y `.firebaserc` fueron creados, el proyecto quedo asociado a `prode-mundial-2026-5b2d9` y el deploy Firestore fue exitoso.

Storage queda diferido porque Firebase Storage requiere upgrade de plan y no es critico para el MVP.

## Objetivo

Dejar definida y validada la seguridad minima de Firebase para que usuarios autenticados solo escriban sus propios pronosticos y solo el admin pueda modificar datos generales del torneo.

## Dependencia estricta

Puede comenzar despues de aprobar el roadmap general. Sin Dia 1 `COMPLETADO`, Dia 2 no puede comenzar.

## Tareas

- Revisar la estructura actual esperada: `users/{uid}`, `prode/mundial2026`, `prode/mundial2026/predictions/{uid}`.
- Definir reglas Firestore para lectura de datos del torneo por usuarios autenticados.
- Definir reglas Firestore para escritura de `predictions/{uid}` solo por `request.auth.uid`.
- Definir criterio admin para escritura de `prode/mundial2026`.
- Definir reglas Storage para `users/{uid}/avatar`.
- Documentar si las reglas se versionan en archivos o se cargan manualmente en Firebase Console.
- Probar registro, login y escritura de perfil/pronosticos.

## Archivos permitidos

- `docs/AVANCE_PROYECTO.md`.
- `docs/FUNCIONALIDADES_PENDIENTES.md`.
- `docs/roadmap-codex/DIA_01_SEGURIDAD_FIREBASE.md`.
- `firestore.rules` si se decide versionar reglas.
- `storage.rules` si se decide versionar reglas.
- `index.html` solo si hace falta ajustar llamadas para cumplir reglas.

## Archivos prohibidos

- `index_backup.html`.
- `index_backup_20260527_1651.html`.
- `prode_mundial_2026_web.html`.
- Imagenes.
- `.git/`.

## Criterios de aceptacion

- Usuario comun puede leer fixture, ranking y su perfil.
- Usuario comun puede escribir solo `prode/mundial2026/predictions/{suUid}`.
- Usuario comun no puede escribir `prode/mundial2026`.
- Usuario comun no puede escribir predicciones de otro UID.
- Admin oficial puede escribir datos generales si el esquema de reglas elegido lo permite.
- Usuario puede subir solo su avatar a `users/{uid}/avatar`.

## Tests obligatorios

- Login con usuario comun.
- Guardar un pronostico del usuario comun.
- Intentar una accion admin con usuario comun y confirmar rechazo o bloqueo.
- Login con `dmcfarlane@prode.local`.
- Guardar/cargar datos generales si el admin esta habilitado.
- Subir avatar propio.

## Resultado esperado

Firebase queda con una base de seguridad real, no dependiente solamente del frontend.

## Resultado de ejecucion

- Se creo `firestore.rules`.
- Se creo `storage.rules`.
- Se ajusto `index.html` para que usuarios comunes no intenten escribir `prode/mundial2026` al agregarse localmente como jugador.
- Se ajusto `index.html` para que solo el admin pueda inicializar `prode/mundial2026` si el documento no existe.
- Se creo `firebase.json`.
- Se creo `.firebaserc`.
- Se asocio el proyecto Firebase `prode-mundial-2026-5b2d9`.
- Se desplegaron reglas Firestore correctamente.

## Pendiente diferido

- Publicar y probar `storage.rules` cuando Firebase Storage este disponible en el plan del proyecto.

## Prompt para iniciar el dia

```text
Ejecuta el Dia 1 - Seguridad Firebase. No modifiques codigo hasta revisar docs/AVANCE_PROYECTO.md, docs/MEMORIA_PROYECTO.md, docs/FUNCIONALIDADES_PENDIENTES.md y la estructura actual de Firebase en index.html. Al final, marca este archivo como COMPLETADO solo si las reglas y pruebas minimas quedan validadas.
```
