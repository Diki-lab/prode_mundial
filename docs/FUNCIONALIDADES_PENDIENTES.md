# Funcionalidades Pendientes

## Indice

- [Bugs criticos](#bugs-criticos)
- [Prioridad alta](#prioridad-alta)
- [Prioridad media](#prioridad-media)
- [Prioridad baja](#prioridad-baja)
- [Mejoras de seguridad](#mejoras-de-seguridad)
- [Mejoras de diseno mobile](#mejoras-de-diseno-mobile)
- [Mejoras del panel admin](#mejoras-del-panel-admin)
- [Mejoras para pronosticos](#mejoras-para-pronosticos)
- [Mejoras para ranking](#mejoras-para-ranking)
- [Ideas futuras](#ideas-futuras)

## Bugs criticos

Rama de trabajo: `fix/user-identity-by-uid`

### Bug 1 — Las fotos de los usuarios no se muestran en ranking y podio

**Causa raiz:** El sistema de avatares identifica a los jugadores por su `displayName` en lugar de su `uid`, y ademas descarta las URLs de Firebase Storage antes de usarlas.

**Ubicaciones en `index.html`:**

- Linea 2634: `normalizeAvatarSettings` siempre fuerza `photoURL: ''`, descartando cualquier URL real de Firebase Storage guardada en `users/{uid}.photoURL`. El campo llega de Firestore pero nunca se pasa al markup.
- Linea 2625-2628: `getLocalAvatarPath(value)` genera una ruta estatica `img/avatars/[slug].jpg` derivada del nombre. Si el nombre cambia, el slug cambia y el archivo local deja de encontrarse.
- Linea 2691-2706: `getAvatarForPlayer(player)` busca el perfil de avatar en `userProfilesByName` usando el `displayName` como clave. Si el nombre en el ranking no coincide exactamente con el nombre registrado, no encuentra ningun avatar.
- Linea 2677-2688: `rememberUserProfile` solo indexa por `displayName` y `email`, nunca por `uid`. No hay forma de buscar el avatar por identidad estable.

**Solucion propuesta:**

1. En `normalizeAvatarSettings`: preservar `photoURL` tal como llega en lugar de limpiarlo a `''`.
2. En `buildAvatarMarkup`: cuando `photoURL` no esta vacio, renderizar `<img src="{photoURL}">` con prioridad sobre `localAvatarPath`.
3. En `rememberUserProfile`: ademas de indexar por `displayName` y `email`, indexar tambien por `uid` cuando el perfil lo incluya.
4. En `getAvatarForPlayer`: aceptar opcionalmente un segundo parametro `uid` y buscar primero por `uid` en `userProfilesByName`, luego por nombre como fallback.
5. En `getPredictionDocsList` y las llamadas a `rememberUserProfile` desde `loadAllPredictions`: pasar el `uid` del documento de prediccion al perfil de avatar para que quede indexado.

---

### Bug 2 — Los puntos se pierden al cambiar el nombre de usuario

**Causa raiz:** Los pronósticos identifican al jugador solo por `player` (string con el `displayName`). Al cambiar el nombre, los pronosticos viejos quedan con el nombre anterior y el ranking los trata como un jugador distinto.

**Ubicaciones en `index.html`:**

- Linea 2801-2810: `normalizePrediction` solo guarda `{ player, matchId, home, away }`. No hay campo `uid`. El nombre es el unico identificador del jugador en cada pronostico.
- Linea 4117-4143: `savePrediction()` crea la entrada con `player = currentProfileName`. Los pronosticos existentes ya tienen `player` seteado con el nombre viejo y `normalizePrediction` los respeta sin pisarlos.
- Linea 3610: `saveCurrentUserPredictions` usa `normalizePredictionList(predictions, currentProfileName)` como fallback, pero las predicciones ya guardadas tienen su propio `player` que tiene precedencia.
- Linea 4540-4554: `getRankingTotals()` agrupa por `prediction.player === player` (comparacion exacta de string). "Carlos" y "Carlos Nuevo" son dos jugadores distintos con puntos parciales cada uno.
- Linea 2835: `getPredictionDocsList` usa `item.displayName || item.email` como fallback de nombre, pero si el array interno ya tiene `player` seteado con el nombre viejo, ese valor se usa directamente.

**Flujo del problema:**

1. Usuario guarda pronosticos con nombre "Carlos" → `prediction.player = "Carlos"` en Firestore.
2. Usuario cambia su `displayName` a "Carlos Nuevo".
3. Pronosticos viejos siguen teniendo `player: "Carlos"` (el campo ya tiene valor, no se pisa).
4. Pronosticos nuevos tienen `player: "Carlos Nuevo"`.
5. El ranking muestra "Carlos" (puntos viejos) y "Carlos Nuevo" (puntos nuevos) como jugadores distintos.

**Solucion propuesta:**

1. En `normalizePrediction`: propagar el campo `uid` si viene en el objeto fuente: `{ player, matchId, home, away, uid }`.
2. En `getPredictionDocsList`: al aplanar las predicciones de cada documento, inyectar el `uid` del documento padre en cada prediccion individual.
3. En `getRankingTotals`: agrupar por `uid` cuando este disponible; caer a agrupacion por `player` solo para datos legacy sin `uid`.
4. En `saveCurrentUserPredictions`: antes de guardar, normalizar todos los `player` del array local al `currentProfileName` actual. Esto garantiza que al guardar despues de un cambio de nombre, todos los pronosticos del usuario queden con el nombre nuevo.
5. Al actualizar `displayName` con `updateDisplayName`: tambien recargar y re-guardar los pronosticos del usuario para propagar el nombre nuevo a todas las entradas existentes en Firestore.

## Prioridad alta

- Activar Email/Password en Firebase Authentication.
- Publicacion final en GitHub Pages. COMPLETADO en Dia 7: `https://diki-lab.github.io/prode_mundial/` responde HTTP 200 OK.
- Publicar reglas de Firestore que exijan usuario autenticado. COMPLETADO: `firestore.rules` fue compilado y desplegado correctamente.
- Publicar reglas de Storage para avatars. DIFERIDO: Firebase Storage requiere upgrade de plan y no es critico para el MVP.
- Probar registro, login y logout desde GitHub Pages. COMPLETADO para login/logout en Dia 5 con validacion navegador; registro real contra Firebase validado con usuarios comunes temporales.
- Verificar que `prode/mundial2026` no pierda datos existentes. COMPLETADO en Dia 7 con lectura real del documento publicado.
- Validar en produccion el guardado de pronosticos por usuario en `prode/mundial2026/predictions/{uid}`. COMPLETADO en Dia 5: escritura real por UID validada con dos usuarios comunes temporales y usuario comun validado en navegador.
- Implementar reglas Firestore o Firebase custom claims para proteger el rol admin.
- Limitar escritura de resultados reales solo a administradores.
- Implementar bloqueo real de pronosticos. COMPLETADO en `index.html` y validado en navegador en Dia 5.
- Implementar bloqueo real de resultados. COMPLETADO en `index.html` y validado en navegador en Dia 5.

## Prioridad media

- Ampliar mensajes de error de Auth segun nuevos codigos detectados en produccion.
- Mejorar estado visual de carga durante carga inicial de sesion y Firestore.
- Agregar confirmacion al cerrar sesion.
- Agregar recuperacion de contrasena.
- Agregar validacion de email mas clara en el formulario.
- Evitar duplicados de jugadores por diferencias de mayusculas, espacios o acentos.
- Guardar fecha de creacion de usuarios en `users/{uid}`.

## Prioridad baja

- Agregar pagina o modal de ayuda.
- Agregar exportacion de pronosticos.
- Agregar historial de cambios.
- Agregar configuracion de perfil mas completa.
- Fotos reales de perfil integradas con Firebase Storage.
- Mejorar textos de bienvenida y estados vacios.

## Mejoras de seguridad

- No confiar en checks de admin hechos solo en frontend.
- Crear documento de roles, por ejemplo `roles/{uid}`, o usar custom claims.
- Usar reglas Firestore para que solo admins reales escriban resultados y bloqueos.
- Reforzar con reglas Firestore que cada usuario escriba solo sus propios pronosticos.
- Validar en reglas que un usuario no pueda modificar pronosticos ajenos.
- Bloquear escritura de resultados para usuarios no admin.
- Bloquear eliminacion de documentos criticos.
- Evitar exponer informacion sensible en el cliente.
- Usar Git como respaldo principal antes de cambios grandes.

## Mejoras de diseno mobile

- Revisar tablas grandes en pantallas chicas. COMPLETADO en Dia 6 con scroll horizontal y anchos minimos responsive.
- Mejorar navegacion por tabs en mobile. COMPLETADO en Dia 6 con tabs horizontales desplazables.
- Ajustar botones del header para evitar saltos o superposiciones. COMPLETADO en Dia 6 con grillas responsive para header.
- Mejorar pantalla de login/registro en celulares bajos. COMPLETADO en Dia 6 con scroll vertical y titulo compacto.
- Revisar inputs numericos de resultados y pronosticos. COMPLETADO en Dia 6 con controles tactiles mas grandes.
- Agregar estados de foco y tactiles mas claros. COMPLETADO parcialmente en Dia 6 con mayor area tactil; los estados visuales de foco se mantienen segun estilos existentes.

## Mejoras del panel admin

- Crear una vista admin dedicada. COMPLETADO parcialmente en la seccion Admin actual.
- Separar gestion de jugadores, resultados y bloqueo de pronosticos. COMPLETADO para controles MVP.
- Mostrar claramente que el admin actual es visual/local hasta que existan reglas y roles reales.
- Agregar confirmaciones antes de borrar jugadores o pronosticos. COMPLETADO para jugadores; pronosticos consolidados quedan solo lectura.
- Agregar filtros para encontrar pronosticos por usuario o partido.
- Mostrar ultimas acciones administrativas.
- Permitir editar datos del fixture con validaciones.
- Agregar indicador visible de usuario admin autenticado.

## Mejoras para pronosticos

- Bloquear pronosticos por fecha/hora de partido.
- Permitir editar pronostico solo antes del bloqueo. COMPLETADO para bloqueo global `arePredictionsLocked`.
- Mostrar progreso de pronosticos cargados por usuario. COMPLETADO para progreso `cargados/total`.
- Agregar vista "mis pronosticos".
- Separar pronosticos en Firestore por usuario y partido.
- Agregar validaciones para resultados negativos o campos incompletos. COMPLETADO para carga de pronosticos.
- Evitar que un usuario pueda cargar pronosticos por otro usuario.

## Mejoras para ranking

- Agregar desempates configurables.
- Agregar ranking por grupo o fase.
- Agregar ranking historico por fecha.
- Mostrar detalle de puntos por partido.
- Agregar exportacion en CSV.
- Agregar filtros por usuario.
- Mejorar presentacion visual de posiciones.
- Evitar duplicacion entre predicciones legacy y subcoleccion. COMPLETADO: subcoleccion como fuente principal, legacy como fallback.

## Estado actualizado - 27 de mayo de 2026

### Completado de forma incremental

- Guardado de pronosticos por usuario autenticado en `prode/mundial2026/predictions/{uid}`.
- Carga de pronosticos propios al iniciar sesion.
- Ranking compatible con la nueva subcoleccion y con el array historico `prode/mundial2026.predictions`.
- Rol admin funcional por email oficial.
- Funcion frontend `isCurrentUserAdmin()`.
- Visualizacion de rol actual en la UI.
- Google Login removido: el sistema usa solamente Email/Password.
- Avatar por usuario guardado en `users/{uid}` y fotos reales en Firebase Storage.
- Podio visual agregado en Inicio con los 3 primeros del ranking.

### Admin oficial

```text
dmcfarlane@prode.local
```

Ese usuario debe quedar como:

- `role: "admin"`
- `isAdmin: true`

Todos los demas usuarios:

- `role: "user"`
- `isAdmin: false`

No usar Gmail personal como admin oficial. No usar password hardcodeada en frontend. El admin debe registrarse o iniciar sesion por Firebase Auth.

La comparacion admin debe hacerse siempre con email autenticado normalizado mediante `trim().toLowerCase()`. La UI debe mostrar el rol calculado desde Firebase Auth y no desde un valor viejo guardado en Firestore.

### Configuracion pendiente en Firebase Console

Mantener habilitado Email/Password:

```text
Authentication -> Sign-in method -> Email/Password
```

Google Provider no se usa en esta app.

Usuarios sugeridos:

```text
nombre@prode.local
```

Admin oficial:

```text
dmcfarlane@prode.local
```

### Pendiente para seguridad real

El rol admin todavia es funcional en frontend. Falta reforzarlo con reglas Firestore o custom claims.

Reglas futuras recomendadas:

- Usuarios autenticados pueden leer datos del torneo.
- Cada usuario solo puede escribir `prode/mundial2026/predictions/{uid}` si `{uid}` coincide con `request.auth.uid`.
- Solo admins reales pueden escribir resultados, bloqueos y datos generales del torneo.

### Pendiente para admin/ranking

La vista admin muestra pronosticos consolidados en modo solo lectura. Queda pendiente una pantalla admin dedicada para revisar, filtrar o corregir pronosticos por usuario sin riesgo de pisar documentos ajenos.

### Avatar y fotos

- Las fotos reales se guardan en Firebase Storage en `users/{uid}/avatar`.
- La URL de descarga se guarda en `users/{uid}.photoURL`.
- No guardar imagenes base64 grandes en Firestore.
- Mantener los campos livianos actuales: `avatarType`, `avatarValue`, `avatarColor` y `photoURL`.
- La UI de `Mi perfil` solo expone adjuntar foto; inicial, emoji y color quedan como fallback interno.

### Podio de Inicio

- El podio usa los datos reales de `getRankingTotals()`.
- Muestra avatar/foto, nombre y puntos para los 3 primeros.
- Los empates se resuelven con la logica actual del ranking: puntos y luego nombre alfabetico.
- Probar visualmente en desktop y mobile comparando contra la pantalla `Ranking`.

## Ideas futuras

- Notificaciones antes del cierre de pronosticos.
- Invitaciones por link.
- Liga privada con codigo de acceso.
- Multiples torneos o ediciones.
- Panel de estadisticas.
- Comparador de pronosticos entre usuarios.
- Modo solo lectura publico para ranking.
- Backend dedicado si el proyecto crece mas alla de GitHub Pages.
