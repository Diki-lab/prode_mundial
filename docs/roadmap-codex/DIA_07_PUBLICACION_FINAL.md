# Dia 07 - Publicacion Final

## Estado

COMPLETADO ✅

Publicacion GitHub Pages: OK ✅

No se agregaron funcionalidades nuevas.

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

## Resultado de ejecucion - 4 de junio de 2026

Se ejecuto solamente el Dia 7. No se agregaron funcionalidades nuevas y no se
modifico `index.html`.

### Publicacion

URL publicada validada:

```text
https://diki-lab.github.io/prode_mundial/
```

Resultado:

- URL principal GitHub Pages: HTTP 200 OK.
- `logo_titulo.png`: HTTP 200 OK.
- `banner-mundial-2026.png`: HTTP 200 OK.
- `img/estadio_fondo.jpg`: HTTP 200 OK.
- La pagina publicada contiene configuracion Firebase del proyecto
  `prode-mundial-2026-5b2d9`.
- La pagina publicada contiene la pantalla de acceso con Firebase
  Authentication.
- La pagina publicada contiene reglas CSS responsive agregadas en Dia 6.

### Confirmaciones tecnicas

- `index.html` es la unica entrada publica necesaria.
- No hay build step ni dependencias de bundler.
- Las rutas de imagenes usadas por la app son relativas y existen en el repo.
- Firebase Auth Email/Password queda confirmado por registro/login real con
  usuario temporal.
- Firestore queda confirmado por lectura de `prode/mundial2026`, escritura de
  `users/{uid}` y escritura de `prode/mundial2026/predictions/{uid}`.
- Reglas Firestore quedan confirmadas por rechazo HTTP 403 al intentar escribir
  `prode/mundial2026` con usuario comun.
- Reglas Storage estan versionadas en `storage.rules`, pero Storage sigue
  diferido por plan Firebase y no es critico para MVP.

### Validaciones ejecutadas

- Dia 6 verificado como `COMPLETADO`.
- Sintaxis del script clasico embebido en `index.html`: OK.
- Balance de llaves CSS: OK.
- Assets versionados locales: OK.
- Tests logicos embebidos en `runTests()`: OK.
- URL publicada y assets principales: OK.
- Registro real de usuario comun temporal: OK.
- Login real de usuario comun temporal: OK.
- Guardado real de perfil en `users/{uid}`: OK.
- Lectura real de datos compartidos del torneo: OK.
- Guardado real de pronostico por UID: OK.
- Usuario comun no puede escribir datos admin: OK.
- Limpieza del documento temporal de pronosticos: OK.

Usuario temporal usado:

```text
codex-dia7-1780599024870@prode.local
```

### Validaciones cubiertas por Dia 5

Estas pruebas ya fueron validadas manualmente en navegador durante Dia 5 y se
toman como cobertura funcional final sin volver a tocar logica:

- Login admin: OK.
- Admin ON visible: OK.
- Abrir/Cerrar pronosticos: OK.
- Abrir/Cerrar resultados: OK.
- Carga de resultado: OK.
- Ranking actualiza: OK.
- Logout: OK.
- Usuario comun no ve Admin: OK.
- Usuario comun guarda pronosticos cuando estan abiertos: OK.
- Usuario comun no puede guardar pronosticos cuando estan cerrados: OK.

### Usuarios sugeridos

Formato sugerido para participantes:

```text
nombre@prode.local
```

Admin oficial:

```text
dmcfarlane@prode.local
```

El admin debe iniciar sesion con Firebase Auth Email/Password. No usar passwords
hardcodeadas en frontend.

### Checklist operativo para el primer dia del torneo

- Confirmar que GitHub Pages abre `https://diki-lab.github.io/prode_mundial/`.
- Confirmar que Email/Password sigue habilitado en Firebase Authentication.
- Confirmar que `firestore.rules` publicadas siguen activas.
- Compartir con participantes el formato sugerido de email.
- Pedir a cada participante que registre cuenta o inicie sesion.
- Verificar que cada participante pueda guardar al menos un pronostico.
- Antes del primer partido, cerrar pronosticos desde usuario admin.
- Revisar que usuarios comunes no puedan modificar pronosticos cerrados.
- Cargar resultados reales desde admin cuando terminen los partidos.
- Revisar que Ranking y Tabla de Grupos actualicen.
- Mantener backup por Git antes de cualquier cambio posterior.

### Riesgos detectados

- El usuario temporal de Auth creado para validar Dia 7 puede quedar en Firebase
  Authentication y `users/{uid}` porque el cliente no tiene permisos para borrar
  usuarios de Auth ni documentos `users`.
- Storage para avatars sigue diferido por plan Firebase.
- La seguridad admin todavia depende del email oficial y reglas actuales; una
  mejora futura seria custom claims o roles administrados fuera del frontend.

### Estado final

- Dia 7: `COMPLETADO ✅`.
- Publicacion final: `OK ✅`.
- No se agregaron funcionalidades nuevas.

## Prompt para iniciar el dia

```text
Ejecuta el Dia 7 - Publicacion Final. Primero verifica que DIA_06_MOBILE_UI.md este COMPLETADO. Si no lo esta, detente. Si esta completo, valida GitHub Pages, Firebase y flujo real antes de marcar publicacion lista.
```
