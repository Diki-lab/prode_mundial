# Dia 07 - Publicacion Final

## Estado

PENDIENTE.

## Objetivo

Preparar y validar la publicacion final en GitHub Pages con checklist operativo para usar el Prode con participantes reales.

## Dependencia estricta

No puede comenzar si `DIA_06_MOBILE_UI.md` no esta marcado como `COMPLETADO`.

## Tareas

- Verificar que Dia 6 este `COMPLETADO`.
- Confirmar que `index.html` es la unica entrada publica necesaria.
- Confirmar que rutas de imagenes son relativas y funcionan en GitHub Pages.
- Confirmar Firebase Auth Email/Password habilitado.
- Confirmar reglas Firestore/Storage publicadas.
- Probar URL publicada.
- Documentar usuarios sugeridos y admin oficial.
- Preparar checklist de operacion para el primer dia del torneo.
- Actualizar docs finales.

## Archivos permitidos

- `docs/AVANCE_PROYECTO.md`.
- `docs/MEMORIA_PROYECTO.md`.
- `docs/FUNCIONALIDADES_PENDIENTES.md`.
- `docs/roadmap-codex/DIA_07_PUBLICACION_FINAL.md`.
- `index.html` solo para fix bloqueante de publicacion.

## Archivos prohibidos

- `index_backup.html`.
- `index_backup_20260527_1651.html`.
- `prode_mundial_2026_web.html`.
- Cambios de arquitectura.
- Cambios funcionales grandes.
- Imagenes salvo fix de ruta estrictamente necesario.

## Criterios de aceptacion

- La app abre desde GitHub Pages.
- Login funciona desde GitHub Pages.
- Usuario comun puede guardar pronostico.
- Admin puede cargar resultado y bloquear pronosticos.
- Ranking actualiza.
- Mobile funciona razonablemente.
- No hay errores criticos en consola.

## Tests obligatorios

- Abrir URL publicada.
- Login usuario comun.
- Guardar pronostico.
- Logout.
- Login admin.
- Cambiar bloqueo.
- Cargar resultado.
- Revisar ranking.
- Revisar mobile.

## Resultado esperado

Version minima funcional publicada y lista para que los participantes usen el Prode.

## Prompt para iniciar el dia

```text
Ejecuta el Dia 7 - Publicacion Final. Primero verifica que DIA_06_MOBILE_UI.md este COMPLETADO. Si no lo esta, detente. Si esta completo, valida GitHub Pages, Firebase y flujo real antes de marcar publicacion lista.
```
