# Avance Del Proyecto

## Indice

- [Resumen](#resumen)
- [Que se reviso](#que-se-reviso)
- [Que se modifico](#que-se-modifico)
- [Firebase Auth](#firebase-auth)
- [Firestore](#firestore)
- [Login y registro](#login-y-registro)
- [Archivos tocados](#archivos-tocados)
- [Fecha del avance](#fecha-del-avance)
- [Proximos pasos inmediatos](#proximos-pasos-inmediatos)

## Resumen

El proyecto Prode Mundial 2026 es una aplicacion web estatica compatible con GitHub Pages. Permite cargar participantes, partidos, pronosticos, resultados reales, tabla de posiciones, ranking y vistas administrativas basicas.

Actualmente el proyecto funciona principalmente desde `index.html`, con persistencia local en `localStorage` y sincronizacion en Firebase/Firestore cuando hay conexion disponible.

## Que se reviso

- Configuracion de Firebase en `index.html`.
- Uso de Firestore para leer y guardar el documento global `prode/mundial2026`.
- Flujo anterior de login local.
- Uso de participantes, partidos, pronosticos, ranking y controles admin.
- Compatibilidad general con hosting estatico tipo GitHub Pages.

## Que se modifico

- Se creo una copia de seguridad de `index.html` como `index_backup.html`.
- Se agrego Firebase Authentication.
- Se reemplazo el login local hardcodeado por login con email y contrasena.
- Se agrego registro de usuarios con nombre visible, email y contrasena.
- Se removio el login con Google. El acceso actual usa solamente Firebase Auth Email/Password.
- Se agrego cierre de sesion con Firebase Auth.
- Se agrego guardado basico de usuarios en Firestore, coleccion `users`.
- Se oculto el Prode si no existe una sesion autenticada.
- Se reforzaron guards de sesion para evitar acciones del Prode sin usuario autenticado.
- Se mejoraron mensajes de error de Auth y logs de Firestore.

## Firebase Auth

Firebase Auth quedo integrado desde CDN, manteniendo compatibilidad con GitHub Pages.

Funciones usadas:

- `getAuth(app)`
- `createUserWithEmailAndPassword`
- `signInWithEmailAndPassword`
- `updateProfile`
- `onAuthStateChanged`
- `signOut`

El estado de sesion se controla con `onAuthStateChanged`. Si no hay usuario, se muestra solo la pantalla de acceso. Si hay usuario, se habilita la aplicacion.

Tambien se agregaron guards de frontend para bloquear acciones sensibles si no existe usuario autenticado. Esto mejora la UX y evita cambios accidentales desde la interfaz, pero la seguridad real debe completarse con reglas de Firestore.

## Firestore

Firestore sigue usando el documento global:

```text
prode/mundial2026
```

Ese documento guarda datos compartidos:

- `players`
- `matches`
- `predictions`
- `isResultLocked`
- `arePredictionsLocked`
- `updatedAt`

Tambien se agrego la coleccion:

```text
users/{uid}
```

Cada usuario autenticado guarda datos basicos:

- `uid`
- `displayName`
- `email`
- `providers`
- `isAdmin`
- `lastLoginAt`
- `updatedAt`

## Login y registro

El login actual usa email y contrasena contra Firebase Auth.

El registro solicita:

- nombre visible
- email
- contrasena

Al registrar un usuario:

1. Se crea la cuenta con Firebase Auth.
2. Se actualiza el `displayName` con `updateProfile`.
3. Se guarda el usuario en Firestore en `users/{uid}`.
4. El usuario entra al Prode si la autenticacion fue correcta.

El login con Google fue removido para evitar confusion con usuarios internos `nombre@prode.local`.

## Archivos tocados

- `index.html`: integracion de Firebase Auth, login, registro, logout y guardado de usuarios.
- `index_backup.html`: copia de seguridad previa a los cambios de Auth.
- `index_backup_20260527_1651.html`: copia de seguridad previa a las mejoras de sesion, Google, Firestore y UX.
- `docs/AVANCE_PROYECTO.md`: documentacion del avance actual.
- `docs/MEMORIA_PROYECTO.md`: memoria tecnica del proyecto.
- `docs/FUNCIONALIDADES_PENDIENTES.md`: backlog funcional y tecnico.

## Fecha del avance

27 de mayo de 2026.

## Proximos pasos inmediatos

- Activar Email/Password en Firebase Authentication.
- Revisar y publicar reglas de Firestore seguras.
- Probar registro real desde GitHub Pages.
- Probar login real desde GitHub Pages.
- Confirmar que `prode/mundial2026` conserva datos existentes.
- Separar datos por usuario en Firestore para evitar pisado de datos.
- Definir un sistema de admin seguro, idealmente con custom claims o un documento de roles.

## Actualizacion 27 de mayo de 2026 - pronosticos por usuario y roles

Se adapto `index.html` para mantener la app como HTML/CSS/JS puro compatible con GitHub Pages, sin PWA y sin framework.

### Nueva estructura Firestore usada

Datos generales compartidos:

```text
prode/mundial2026
```

Debe contener datos compartidos del torneo:

- `players`
- `matches`
- `isResultLocked`
- `arePredictionsLocked`
- `updatedAt`

Pronosticos por usuario autenticado:

```text
prode/mundial2026/predictions/{uid}
```

Cada documento incluye:

- `uid`
- `displayName`
- `email`
- `predictions`
- `createdAt`
- `updatedAt`

Usuarios:

```text
users/{uid}
```

Cada documento incluye:

- `uid`
- `displayName`
- `email`
- `role`
- `isAdmin`
- `providers`
- `avatarType`
- `avatarValue`
- `avatarColor`
- `photoURL`
- `createdAt`
- `updatedAt`
- `lastLoginAt`

### Roles

El admin oficial es:

```text
dmcfarlane@prode.local
```

Ese email queda con:

- `role: "admin"`
- `isAdmin: true`

Todo otro usuario queda con:

- `role: "user"`
- `isAdmin: false`

No se usa Gmail personal como admin oficial. Tampoco se debe volver a un login hardcodeado con usuario y password en el frontend: el admin debe registrarse o iniciar sesion mediante Firebase Auth igual que cualquier otro usuario.

La comparacion de rol se hace siempre contra el email autenticado normalizado con `trim().toLowerCase()`. Si `users/{uid}` ya existia con `role: "user"` para `dmcfarlane@prode.local`, se actualiza automaticamente al iniciar sesion porque `saveUserProfile(user)` vuelve a escribir `role` e `isAdmin` con `merge: true`.

La UI usa `isCurrentUserAdmin()` para mostrar u ocultar controles admin. Esta proteccion es funcional en frontend, pero no reemplaza reglas Firestore ni custom claims.

### Pronosticos

Al iniciar sesion se cargan:

1. Datos generales desde `prode/mundial2026`.
2. Pronosticos del usuario actual desde `prode/mundial2026/predictions/{uid}`.
3. Pronosticos de todos los usuarios para ranking desde la subcoleccion `predictions`.

Si un usuario todavia no tiene documento de pronosticos, la app inicia con pronosticos vacios. Para compatibilidad, si existen pronosticos historicos en `prode/mundial2026.predictions`, se usan para ranking y como recuperacion inicial del usuario actual cuando coincide el nombre.

Al guardar, la app guarda solo los pronosticos del usuario autenticado en `prode/mundial2026/predictions/{uid}` con `setDoc(..., { merge: true })`.

### Ranking

El ranking ahora intenta leer la subcoleccion:

```text
prode/mundial2026/predictions
```

Tambien mantiene compatibilidad con el array historico `prode/mundial2026.predictions`. La pantalla de ranking no se elimina ni cambia de estructura visual.

La vista admin muestra los pronosticos consolidados en modo solo lectura para evitar borrar o pisar datos de otros usuarios desde una implementacion parcial.

### Login

Google Login fue removido. El sistema usa solamente Firebase Auth con Email/Password.

Usuarios sugeridos:

```text
nombre@prode.local
```

Admin oficial:

```text
dmcfarlane@prode.local
```

### Seguridad pendiente

La seguridad real todavia debe reforzarse con reglas Firestore o custom claims. Recomendacion futura:

- Permitir a usuarios autenticados leer datos generales.
- Permitir que cada usuario escriba solo `prode/mundial2026/predictions/{uid}` donde `{uid}` sea su propio UID.
- Permitir resultados, bloqueos y cambios administrativos solo a admins reales.
- No confiar en `isAdmin` del frontend como unica barrera de seguridad.

### Backups

No se creo ningun backup nuevo de `index.html`. El respaldo principal es Git.

## Actualizacion - avatar por usuario

Se agrego foto real de perfil por usuario. El fallback liviano se mantiene internamente en Firestore, pero en `Mi perfil` el usuario solo edita la foto real. Las fotos se guardan en Firebase Storage, sin guardar imagenes base64 grandes en Firestore.

Los campos se guardan en:

```text
users/{uid}
```

Campos de avatar:

- `avatarType`: `"initial"`, `"emoji"` o `"color"`
- `avatarValue`: texto corto para inicial o emoji
- `avatarColor`: color hexadecimal del fondo
- `photoURL`: opcional, si Firebase Auth lo provee
- `updatedAt`

`avatarType`, `avatarValue` y `avatarColor` quedan como fallback interno cuando no hay foto. La UI visible de `Mi perfil` muestra solamente `Adjuntar foto`, vista previa y `Guardar avatar`.

Fotos reales:

```text
Storage: users/{uid}/avatar
```

Al iniciar sesion, la app carga `users/{uid}` y aplica el avatar en el header, en la pantalla `Mi perfil` y en el podio de Inicio cuando corresponde. Si no existe avatar guardado, usa inicial del nombre visible, inicial del email o icono por defecto.

El avatar se limpia visualmente al cerrar sesion para evitar mostrar datos del usuario anterior.

Para probar la subida de foto:

1. Iniciar sesion con Email/Password.
2. Abrir `Mi perfil`.
3. Adjuntar una imagen menor a 2 MB o permitir que la app intente reducirla.
4. Presionar `Guardar avatar`.
5. Verificar que la foto se vea en header, perfil y que `users/{uid}.photoURL` quede actualizado.
6. Cerrar sesion y volver a iniciar sesion para confirmar que la foto se recupera desde Firestore/Storage.

## Actualizacion - ranking destacado en Inicio

La pantalla Inicio ahora incluye un podio visual con los 3 primeros del ranking actual. Usa la misma logica de `getRankingTotals()`: ordena por puntos descendente y, ante empate, por nombre alfabetico.

Cada tarjeta del podio muestra:

1. Avatar o foto circular.
2. Nombre del jugador.
3. Puntos.

El ranking completo sigue existiendo en su pantalla original.

Para probar el podio:

1. Cargar resultados reales o usar datos con puntos existentes.
2. Abrir `Inicio`.
3. Confirmar que se muestran hasta 3 jugadores con avatar/foto, nombre y puntos.
4. Comparar contra la pantalla `Ranking`.

## Actualizacion 3 de junio de 2026 - Dia 1 Seguridad Firebase

Se ejecuto el Dia 1 del roadmap Codex a nivel repositorio.

Archivos agregados:

- `firestore.rules`
- `storage.rules`

Cambios funcionales minimos:

- `index.html` ya no intenta guardar `prode/mundial2026` cuando un usuario comun se agrega localmente a la lista de jugadores al iniciar sesion.
- Si `prode/mundial2026` no existe, solo el admin puede inicializarlo desde la app.

Reglas Firestore versionadas:

- Usuarios autenticados pueden leer `prode/mundial2026`.
- Solo `dmcfarlane@prode.local` puede crear o actualizar `prode/mundial2026`.
- Usuarios autenticados pueden leer perfiles y predicciones.
- Cada usuario solo puede crear o actualizar `prode/mundial2026/predictions/{uid}` cuando `{uid}` coincide con `request.auth.uid`.
- Los usuarios pueden escribir su propio `users/{uid}` con rol coherente con el email autenticado.

Reglas Storage versionadas:

- Usuarios autenticados pueden leer avatars.
- Cada usuario solo puede crear, actualizar o borrar `users/{uid}/avatar` cuando `{uid}` coincide con `request.auth.uid`.
- La subida queda limitada a imagenes menores a 2 MB.

Estado:

- Dia 1 fue validado y marcado como `COMPLETADO`.
- `firestore.rules` fue compilado y desplegado correctamente en Firebase.
- `firebase.json` y `.firebaserc` fueron creados.
- El proyecto quedo asociado a `prode-mundial-2026-5b2d9`.
- Storage queda diferido porque Firebase Storage requiere upgrade de plan y no es critico para el MVP.

## Actualizacion 3 de junio de 2026 - Dia 2 Bloqueos

Se ejecuto el Dia 2 del roadmap Codex.

Cambios en `index.html`:

- Se agrego bloqueo real de pronosticos en `savePrediction()`.
- Se agrego bloqueo real de borrado de pronosticos en `deletePrediction()`.
- Se agrego bloqueo real de resultados en `updateResult()`.
- Cuando `arePredictionsLocked` esta activo, los inputs y el boton de pronosticos quedan deshabilitados.
- Cuando `isResultLocked` esta activo, los inputs de resultados quedan deshabilitados.
- Se agregaron mensajes visibles de estado en Fixture, Pronosticos y Administracion.
- Se agregaron controles admin para abrir/cerrar pronosticos y bloquear/desbloquear resultados.
- Se agregaron tests logicos embebidos para helpers de bloqueo.

Funciones nuevas:

- `isPredictionEditingBlocked()`
- `isResultEditingBlocked()`
- `getPredictionLockText()`
- `getResultLockText()`
- `renderLockStatuses()`

Funciones modificadas:

- `updatePredictionUserStatus()`
- `renderFixture()`
- `updateResult()`
- `savePrediction()`
- `deletePrediction()`
- `togglePredictionLock()`
- `renderAll()`
- `runTests()`

Validacion:

- Sintaxis de scripts embebidos en `index.html`: OK.
- Tests logicos de bloqueo agregados a `runTests()`: OK a nivel sintaxis/logica local.

Pendiente operativo:

- Ejecutar pruebas manuales en navegador con usuario comun y admin para confirmar persistencia real de los flags en Firestore.

Pruebas reales reportadas:

- Usuario comun no puede guardar pronosticos cuando estan bloqueados.
- Usuario comun puede guardar pronosticos cuando estan abiertos.
- Admin puede abrir/cerrar pronosticos.
- Admin puede abrir/cerrar resultados.
- Admin no puede cargar resultados si estan bloqueados.
- Admin puede cargar resultados si estan abiertos.
- No hubo errores visibles en consola.

Bugfix posterior:

- Se agregaron controles visibles en Fixture para que el admin pueda abrir/cerrar pronosticos y abrir/cerrar resultados desde la misma pantalla donde carga marcadores.
- Se unificaron textos de botones como `Abrir/Cerrar pronosticos` y `Abrir/Cerrar resultados`.
- `toggleResultLock()` ahora muestra confirmacion del nuevo estado luego de cambiar resultados.

## Actualizacion 3 de junio de 2026 - Dia 3 Admin Panel

Se ejecuto el Dia 3 del roadmap Codex.

Cambios en `index.html`:

- Se oculto el acceso Admin del header para usuarios comunes.
- Se bloqueo `showTab('jugadores')` para usuarios no admin.
- Se agrego estado visible de identidad admin con email y rol.
- Se agrego acceso rapido desde Admin hacia Fixture para cargar resultados.
- Se agregaron controles de bloqueo tambien en Fixture para evitar que el admin quede sin forma visible de abrir resultados bloqueados.
- Se agrego confirmacion antes de borrar jugadores.
- Se agrego validacion basica contra duplicados de jugadores por espacios y mayusculas/minusculas.
- Se reforzo que los pronosticos consolidados en la vista admin sean solo lectura.

Funciones nuevas:

- `normalizePlayerName()`
- `playerExists()`

Funciones modificadas:

- `showTab()`
- `renderLockStatuses()`
- `updateAdminUI()`
- `deletePrediction()`
- `deletePlayer()`
- `addPlayer()`
- `runTests()`

Validacion:

- Sintaxis de scripts embebidos en `index.html`: OK.
- Tests logicos de normalizacion/deteccion de duplicados agregados a `runTests()`.

Pendiente operativo:

- Ejecutar pruebas manuales en navegador con usuario comun y admin para confirmar visibilidad del panel, agregado de jugador, duplicados y confirmacion de borrado.

Pruebas reales reportadas:

- Usuario comun no ve acceso Admin.
- Usuario comun no puede entrar a jugadores/admin.
- Admin ve panel admin.
- Admin ve email/rol.
- Admin tiene acceso a carga de resultados.
- Agregar jugador duplicado muestra aviso.
- Borrar jugador pide confirmacion.
- Pronosticos consolidados quedan solo lectura.

## Actualizacion 3 de junio de 2026 - Dia 4 Ranking y Pronosticos

Se ejecuto el Dia 4 del roadmap Codex.

Cambios en `index.html`:

- Se agrego deduplicacion de pronosticos por jugador y partido.
- El ranking usa como fuente principal la subcoleccion `prode/mundial2026/predictions`.
- El array legacy `prode/mundial2026.predictions` queda solo como fallback si no hay documentos en la subcoleccion.
- Se mantiene fallback local si no hay subcoleccion ni legacy.
- Se mejoro la validacion de inputs para evitar guardar campos vacios como cero.
- Se agrego progreso visible en la pantalla de pronosticos.
- El perfil muestra `Mis pronosticos` como progreso `cargados/total`.
- La vista admin muestra cantidad consolidada de pronosticos unicos.

Funciones nuevas:

- `getPredictionKey()`
- `dedupePredictionList()`
- `getPredictionDocsList()`
- `getRankingPredictionSource()`
- `getPredictionProgress()`

Funciones modificadas:

- `updatePredictionUserStatus()`
- `applyProfile()`
- `loadCloudData()`
- `saveCurrentUserPredictions()`
- `savePrediction()`
- `renderPredictions()`
- `getRankingTotals()`
- `runTests()`

Validacion:

- Sintaxis de scripts embebidos en `index.html`: OK.
- Tests logicos de deduplicacion, prioridad de subcoleccion y progreso agregados a `runTests()`.

Pendiente operativo:

- Ejecutar pruebas manuales en navegador con al menos dos usuarios para confirmar escritura por UID, ranking consolidado y ausencia de duplicados.

## Actualizacion 4 de junio de 2026 - Dia 5 Test Flujo Completo

Se ejecuto solamente el Dia 5 del roadmap Codex. No se implementaron
funcionalidades nuevas, no se modifico diseno y no se avanzo al Dia 6.

Validacion local:

- Sintaxis del script clasico embebido en `index.html`: OK.
- Tests logicos embebidos en `runTests()`: OK.

Validacion real contra Firebase con usuarios comunes temporales:

- Registro real de dos usuarios comunes: OK.
- Login real de ambos usuarios comunes: OK.
- Escritura de perfil en `users/{uid}`: OK.
- Lectura de `prode/mundial2026` con usuario autenticado: OK.
- Escritura de pronosticos separados por UID en
  `prode/mundial2026/predictions/{uid}`: OK.
- Lectura de la subcoleccion `predictions` para ranking consolidado: OK.
- Verificacion de que ambos usuarios aparecen como fuente de ranking: OK.
- Intento de escritura admin sobre `prode/mundial2026` con usuario comun:
  rechazado por Firestore con HTTP 403.
- Se limpiaron los documentos de pronosticos temporales creados durante la
  prueba.

Usuarios temporales usados:

```text
codex-dia5-a-1780596617541@prode.local
codex-dia5-b-1780596617541@prode.local
```

Validacion manual en navegador reportada:

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

Estado:

- Dia 5 queda como `COMPLETADO ✅`.
- Validacion navegador: `OK ✅`.
- No se encontraron bugs bloqueantes en las pruebas ejecutadas.
- `index.html` no fue modificado.
- No se avanza al Dia 6.

## Actualizacion 4 de junio de 2026 - Dia 6 Mobile UI

Se ejecuto solamente el Dia 6 del roadmap Codex. No se modifico logica
Firebase, ranking, pronosticos ni roles admin. No se avanzo al Dia 7.

Cambios en `index.html`:

- Se reforzo el header para tablet con areas de grilla separadas para marca,
  usuario y navegacion.
- Se mejoro el header mobile para evitar superposicion de logo, usuario,
  acciones superiores y accesos principales.
- Se mantuvo visible la informacion del usuario en mobile con truncado seguro
  para nombres, email y rol.
- Se cambio la navegacion secundaria mobile a tabs horizontales desplazables.
- Se aumento el alto minimo y tamano tactil de botones principales.
- Se ajustaron tablas para scroll horizontal y anchos minimos mas razonables.
- Se apilaron controles admin en mobile.
- Se ajusto login/registro para pantallas chicas con scroll vertical y titulo
  mas compacto.
- Se agregaron reglas especificas para pantallas de hasta 420px.

Validacion:

- Sintaxis del script clasico embebido en `index.html`: OK.
- Balance de llaves CSS: OK.
- Tests logicos embebidos en `runTests()`: OK.
- Revision estatica de reglas para 360px, 768px y desktop: OK.

Riesgos:

- No hubo navegador interactivo disponible para capturas reales a 360px, 768px y
  desktop, por lo que puede requerir ajuste fino visual posterior.
- Las tablas siguen usando scroll horizontal en mobile cuando tienen muchas
  columnas; se mantiene asi para no cambiar estructura ni logica.

Estado:

- Dia 6 queda como `COMPLETADO ✅`.
- No se avanza al Dia 7.

## Actualizacion 4 de junio de 2026 - Dia 7 Publicacion Final

Se ejecuto solamente el Dia 7 del roadmap Codex. No se agregaron
funcionalidades nuevas y no se modifico `index.html`.

Publicacion:

- URL publicada validada: `https://diki-lab.github.io/prode_mundial/`.
- URL principal GitHub Pages: HTTP 200 OK.
- `logo_titulo.png`: HTTP 200 OK.
- `banner-mundial-2026.png`: HTTP 200 OK.
- `img/estadio_fondo.jpg`: HTTP 200 OK.
- La pagina publicada contiene configuracion Firebase del proyecto
  `prode-mundial-2026-5b2d9`.
- La pagina publicada contiene la pantalla de acceso con Firebase
  Authentication.
- La pagina publicada contiene reglas CSS responsive agregadas en Dia 6.

Validacion tecnica:

- `index.html` es la unica entrada publica necesaria.
- Las rutas de imagenes son relativas y existen en el repo.
- Sintaxis del script clasico embebido en `index.html`: OK.
- Balance de llaves CSS: OK.
- Tests logicos embebidos en `runTests()`: OK.
- Registro/login real de usuario comun temporal contra Firebase Auth: OK.
- Lectura real de `prode/mundial2026`: OK.
- Escritura real de `users/{uid}`: OK.
- Escritura real de `prode/mundial2026/predictions/{uid}`: OK.
- Intento de escritura admin con usuario comun rechazado por Firestore con HTTP
  403: OK.

Usuario temporal usado:

```text
codex-dia7-1780599024870@prode.local
```

Checklist operativo definido:

- Confirmar URL publicada antes de compartir.
- Confirmar Email/Password habilitado en Firebase Authentication.
- Confirmar reglas Firestore publicadas.
- Compartir formato sugerido `nombre@prode.local`.
- Verificar que cada participante pueda guardar al menos un pronostico.
- Cerrar pronosticos desde admin antes del primer partido.
- Cargar resultados desde admin al terminar partidos.
- Revisar Ranking y Tabla de Grupos despues de cargar resultados.

Riesgos:

- El usuario temporal creado para validar Dia 7 puede quedar en Firebase Auth y
  `users/{uid}` porque el cliente no puede borrar usuarios de Auth ni documentos
  `users`.
- Storage para avatars sigue diferido por plan Firebase.
- Una mejora futura de seguridad seria custom claims o roles administrados fuera
  del frontend.

Estado:

- Dia 7 queda como `COMPLETADO ✅`.
- Publicacion final: `OK ✅`.
