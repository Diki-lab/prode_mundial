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
- Se agrego cierre de sesion con Firebase Auth.
- Se agrego guardado basico de usuarios en Firestore, coleccion `users`.
- Se oculto el Prode si no existe una sesion autenticada.

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

## Archivos tocados

- `index.html`: integracion de Firebase Auth, login, registro, logout y guardado de usuarios.
- `index_backup.html`: copia de seguridad previa a los cambios de Auth.
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
