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
- Mantener Email/Password y sumar Google Auth por popup, sin cambiar `firebaseConfig`.
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

El usuario accede con email y contrasena usando Firebase Authentication. Tambien existe acceso con Google mediante popup, siempre que el proveedor Google este habilitado en Firebase Console y el dominio de GitHub Pages este autorizado.

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
- Para Google Auth, confirmar que el dominio de GitHub Pages este autorizado en Firebase Authentication.
