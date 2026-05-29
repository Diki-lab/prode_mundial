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

Se agrego personalizacion de avatar por usuario sin usar Firebase Storage y sin guardar imagenes base64 grandes en Firestore.

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

Al iniciar sesion, la app carga `users/{uid}` y aplica el avatar en el header y en la pantalla `Mi perfil`. Si no existe avatar guardado, usa inicial del nombre visible, inicial del email o icono por defecto.

El avatar se limpia visualmente al cerrar sesion para evitar mostrar datos del usuario anterior.
