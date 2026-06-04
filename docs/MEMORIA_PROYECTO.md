# Memoria Del Proyecto

## Indice

- [Nombre del proyecto](#nombre-del-proyecto)
- [Objetivo](#objetivo)
- [Estructura general](#estructura-general)
- [Tecnologias usadas](#tecnologias-usadas)
- [Datos importantes](#datos-importantes)
- [Decisiones tomadas](#decisiones-tomadas)
- [Advertencias para futuros cambios](#advertencias-para-futuros-cambios)
- [Como funciona el login](#como-funciona-el-login)
- [Como funciona Firestore](#como-funciona-firestore)
- [Como trabajar sin romper GitHub Pages](#como-trabajar-sin-romper-github-pages)

## Nombre del proyecto

Prode Mundial 2026.

## Objetivo

Crear una aplicacion web para organizar un Prode del Mundial 2026, permitiendo que usuarios registren pronosticos, consulten partidos, vean ranking, revisen tablas de grupos y que un administrador cargue resultados reales.

## Estructura general

El proyecto esta organizado como una web estatica:

```text
/
  index.html
  index_backup.html
  prode_mundial_2026_web.html
  PLAN_BACKEND_PRODE.md
  banner-mundial-2026.png
  logo_titulo.png
  img/
  docs/
```

La aplicacion principal esta concentrada en `index.html`. No hay build step, servidor Node ni framework frontend. Esto facilita publicacion directa en GitHub Pages.

## Tecnologias usadas

- HTML.
- CSS embebido.
- JavaScript vanilla.
- Firebase App SDK desde CDN.
- Firebase Authentication desde CDN.
- Firebase Firestore desde CDN.
- Firebase Storage desde CDN para fotos de perfil.
- `localStorage` para persistencia local.
- GitHub Pages como hosting estatico esperado.

## Datos importantes

Firebase esta configurado con el proyecto:

```text
prode-mundial-2026-5b2d9
```

Documento Firestore compartido:

```text
prode/mundial2026
```

Coleccion de usuarios:

```text
users/{uid}
```

Claves locales principales:

- `players2026`
- `matches2026`
- `predictions2026`
- `lockResults2026`
- `lockPredictions2026`
- `theme2026`
- `profileName2026`
- `profilePhoto2026`

## Decisiones tomadas

- Mantener la app como archivo estatico para conservar compatibilidad con GitHub Pages.
- Usar Firebase Auth directamente desde CDN, sin bundler.
- Usar `onAuthStateChanged` como fuente de verdad para mostrar u ocultar el Prode.
- Mantener Firebase Auth Email/Password, sin Google Login y sin cambiar `firebaseConfig`.
- Mantener el documento existente `prode/mundial2026` para no borrar datos actuales.
- Guardar datos basicos de usuario en `users/{uid}` sin afectar la estructura principal.
- Conservar las funciones actuales de fixture, pronosticos, ranking y admin.
- Preparar el campo `isAdmin` en `users/{uid}` como base futura, aunque el control admin real todavia no depende de reglas.

## Advertencias para futuros cambios

- El documento `prode/mundial2026` es global. Si dos usuarios escriben al mismo tiempo, puede haber pisado de datos.
- El panel admin todavia depende de logica del frontend. Para seguridad real debe moverse a reglas Firestore con roles o custom claims.
- El campo `isAdmin` guardado en `users/{uid}` es informativo/preparatorio; no debe considerarse seguridad por si solo.
- No borrar ni recrear `prode/mundial2026` sin backup previo.
- No introducir dependencias que requieran build si se quiere seguir publicando directo en GitHub Pages.
- Cualquier cambio en IDs HTML puede romper funciones que usan `onclick` inline.
- Las reglas de Firestore deben acompanarse con la estructura de datos; reglas seguras son dificiles si todo se guarda en un unico documento global.

## Como funciona el login

El usuario accede con email y contrasena usando Firebase Authentication. Google Login fue removido porque el sistema usa usuarios internos con formato `nombre@prode.local`.

Flujo:

1. La pantalla de login se muestra por defecto.
2. Firebase inicializa Auth.
3. `onAuthStateChanged` detecta si hay usuario autenticado.
4. Si no hay usuario, `header` y `main` quedan ocultos.
5. Si hay usuario, se oculta el login y se muestra la aplicacion.
6. El nombre visible se toma de `user.displayName` o del prefijo del email.
7. La app muestra nombre, email y puntos del usuario autenticado.
8. El boton `Salir` ejecuta `signOut`.

Las acciones sensibles usan un guard de frontend (`requireAuth`) para no operar sin sesion. Este guard no reemplaza reglas de Firestore.

## Como funciona Firestore

La app carga datos compartidos desde:

```text
prode/mundial2026
```

Cuando hay datos en la nube, reemplaza el estado local con los arrays y flags de Firestore. Cuando no existe el documento, inicializa Firestore con los datos locales.

Los cambios de la app ejecutan `saveAll()`, que guarda en `localStorage` y despues intenta sincronizar con Firestore.

Los datos de usuario autenticado se guardan aparte:

```text
users/{uid}
```

Esto permite tener identidad Firebase sin cambiar todavia toda la estructura de pronosticos.

Los documentos `users/{uid}` incluyen datos basicos de sesion, proveedores de Auth y un campo `isAdmin` preparatorio. El guardado usa `merge: true` para no pisar otros campos que se agreguen en el futuro.

## Como trabajar sin romper GitHub Pages

- Mantener rutas relativas para imagenes y archivos.
- Evitar imports locales que requieran servidor o bundler.
- Usar SDKs por CDN si se agregan servicios Firebase.
- Mantener proveedores de Auth via CDN y evitar librerias que requieran empaquetado.
- No depender de variables de entorno en runtime.
- Probar abriendo `index.html` y tambien desde GitHub Pages.
- No mover `index.html` fuera de la raiz si GitHub Pages publica desde root.
- Evitar APIs que no funcionen en HTTPS publico o navegadores modernos.
- Para Auth, mantener habilitado Email/Password en Firebase Authentication.

## Publicacion final

URL publicada:

```text
https://diki-lab.github.io/prode_mundial/
```

Estado al 4 de junio de 2026:

- GitHub Pages responde HTTP 200 OK.
- `index.html` sigue siendo la unica entrada publica necesaria.
- Las imagenes principales responden correctamente desde GitHub Pages:
  `logo_titulo.png`, `banner-mundial-2026.png` e `img/estadio_fondo.jpg`.
- Firebase usa el proyecto `prode-mundial-2026-5b2d9`.
- Email/Password esta validado con registro/login real de usuario temporal.
- Firestore esta validado con lectura del torneo, guardado de perfil y guardado
  de pronostico por UID.
- Las reglas rechazan escritura admin desde usuario comun.

Checklist operativo recomendado:

- Confirmar que la URL publicada abre antes de compartirla.
- Confirmar que Email/Password sigue habilitado en Firebase Authentication.
- Confirmar que reglas Firestore siguen publicadas.
- Usar emails internos con formato `nombre@prode.local`.
- Mantener como admin oficial a `dmcfarlane@prode.local`.
- Antes del primer partido, cerrar pronosticos desde el usuario admin.
- Cargar resultados reales desde admin al terminar cada partido.
- Revisar Ranking y Tabla de Grupos despues de cargar resultados.
- Usar Git como backup antes de cambios posteriores.

## Estado actual actualizado - 27 de mayo de 2026

La app sigue siendo una web estatica en `index.html`. No se convirtio en PWA, no se agrego framework y no se agrego build step.

### Auth

Firebase Auth funciona con:

- Registro Email/Password.
- Login Email/Password.
- Google Login removido.
- Logout.
- `onAuthStateChanged` como fuente de verdad de sesion.

El Prode se mantiene oculto si no hay sesion autenticada.

### Firestore

La estructura actual queda asi:

```text
users/{uid}
prode/mundial2026
prode/mundial2026/predictions/{uid}
```

`prode/mundial2026` queda para datos generales del torneo. Los pronosticos nuevos ya no deben guardarse como array global dentro de ese documento.

`prode/mundial2026/predictions/{uid}` guarda los pronosticos del usuario autenticado:

- `uid`
- `displayName`
- `email`
- `predictions`
- `createdAt`
- `updatedAt`

### Usuarios y roles

`users/{uid}` se actualiza con:

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

Regla funcional de frontend:

```text
dmcfarlane@prode.local -> admin
otros usuarios registrados -> user
```

No se usa Gmail personal como admin oficial. No debe existir password hardcodeada en el frontend ni se debe volver al login local anterior. El usuario admin debe existir e iniciar sesion por Firebase Auth.

El rol se calcula desde el email autenticado normalizado con `trim().toLowerCase()`. La UI no debe depender de un valor viejo de Firestore para mostrar el rol. En cada inicio de sesion, `users/{uid}` se actualiza con el rol calculado, corrigiendo documentos previos que hubieran quedado como `user`.

La funcion principal del frontend para admin es:

```text
isCurrentUserAdmin()
```

Advertencia: esto mejora la UX y la separacion funcional, pero no es seguridad fuerte. Para seguridad real hay que usar reglas Firestore o custom claims.

### Ranking

El ranking usa los pronosticos leidos desde la subcoleccion `prode/mundial2026/predictions`. Para no romper datos existentes, tambien contempla el array historico `prode/mundial2026.predictions` si todavia existe.

### Login

Google Login fue removido. La app usa solamente Firebase Auth Email/Password.

Usuarios sugeridos:

```text
nombre@prode.local
```

Admin oficial:

```text
dmcfarlane@prode.local
```

### Backups

No crear mas archivos `index_backup_YYYYMMDD_HHMM.html`. El proyecto esta versionado en Git y Git debe usarse como respaldo principal.

### Avatar de usuario

El avatar de cada usuario se guarda en Firestore dentro de:

```text
users/{uid}
```

Campos usados:

- `avatarType`: `"initial"`, `"emoji"` o `"color"`
- `avatarValue`: valor corto para mostrar en el circulo
- `avatarColor`: color de fondo
- `photoURL`: opcional si Firebase Auth entrega una URL
- `updatedAt`

Las fotos reales de perfil se guardan en Firebase Storage:

```text
users/{uid}/avatar
```

La URL descargable se guarda en `users/{uid}.photoURL`. No se guardan imagenes base64 grandes en Firestore. Para usuarios internos `nombre@prode.local`, si no hay foto se muestra inicial, emoji o color como fallback interno. La UI de `Mi perfil` solo permite adjuntar y guardar una foto real.

### Ranking destacado en Inicio

La pantalla Inicio muestra un podio visual con los 3 primeros del ranking. El calculo usa `getRankingTotals()`, igual que el ranking completo:

- Mas puntos primero.
- Si hay empate, orden alfabetico por nombre.

Cada tarjeta muestra avatar/foto, nombre y puntos, en ese orden. El ranking completo se mantiene en la pantalla `Ranking`.

Pruebas recomendadas:

- Subir una foto desde `Mi perfil`, guardar y confirmar que aparece en header y perfil.
- Cerrar sesion, volver a ingresar y confirmar que la foto se carga desde `users/{uid}.photoURL`.
- Abrir `Inicio` y comparar el podio contra el ranking completo.
