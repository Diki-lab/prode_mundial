# Funcionalidades Pendientes

## Indice

- [Prioridad alta](#prioridad-alta)
- [Prioridad media](#prioridad-media)
- [Prioridad baja](#prioridad-baja)
- [Mejoras de seguridad](#mejoras-de-seguridad)
- [Mejoras de diseno mobile](#mejoras-de-diseno-mobile)
- [Mejoras del panel admin](#mejoras-del-panel-admin)
- [Mejoras para pronosticos](#mejoras-para-pronosticos)
- [Mejoras para ranking](#mejoras-para-ranking)
- [Ideas futuras](#ideas-futuras)

## Prioridad alta

- Activar Email/Password en Firebase Authentication.
- Activar Google en Firebase Authentication y autorizar el dominio de GitHub Pages.
- Publicar reglas de Firestore que exijan usuario autenticado.
- Probar registro, login y logout desde GitHub Pages.
- Probar login con Google desde GitHub Pages.
- Verificar que `prode/mundial2026` no pierda datos existentes.
- Separar pronosticos por usuario para evitar pisado de datos.
- Implementar roles admin reales en Firestore o Firebase custom claims.
- Limitar escritura de resultados reales solo a administradores.

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
- Agregar avatar remoto en vez de guardar imagen base64 en `localStorage`.
- Mejorar textos de bienvenida y estados vacios.

## Mejoras de seguridad

- No confiar en checks de admin hechos solo en frontend.
- Crear documento de roles, por ejemplo `roles/{uid}`, o usar custom claims.
- Usar reglas Firestore para que solo admins reales escriban resultados y bloqueos.
- Reestructurar Firestore para que cada usuario escriba solo sus propios pronosticos.
- Validar en reglas que un usuario no pueda modificar pronosticos ajenos.
- Bloquear escritura de resultados para usuarios no admin.
- Bloquear eliminacion de documentos criticos.
- Evitar exponer informacion sensible en el cliente.
- Agregar backups manuales o exportaciones antes de cambios grandes.

## Mejoras de diseno mobile

- Revisar tablas grandes en pantallas chicas.
- Mejorar navegacion por tabs en mobile.
- Ajustar botones del header para evitar saltos o superposiciones.
- Mejorar pantalla de login/registro en celulares bajos.
- Revisar inputs numericos de resultados y pronosticos.
- Agregar estados de foco y tactiles mas claros.

## Mejoras del panel admin

- Crear una vista admin dedicada.
- Separar gestion de jugadores, resultados y bloqueo de pronosticos.
- Mostrar claramente que el admin actual es visual/local hasta que existan reglas y roles reales.
- Agregar confirmaciones antes de borrar jugadores o pronosticos.
- Agregar filtros para encontrar pronosticos por usuario o partido.
- Mostrar ultimas acciones administrativas.
- Permitir editar datos del fixture con validaciones.
- Agregar indicador visible de usuario admin autenticado.

## Mejoras para pronosticos

- Bloquear pronosticos por fecha/hora de partido.
- Permitir editar pronostico solo antes del bloqueo.
- Mostrar progreso de pronosticos cargados por usuario.
- Agregar vista "mis pronosticos".
- Separar pronosticos en Firestore por usuario y partido.
- Agregar validaciones para resultados negativos o campos incompletos.
- Evitar que un usuario pueda cargar pronosticos por otro usuario.

## Mejoras para ranking

- Agregar desempates configurables.
- Agregar ranking por grupo o fase.
- Agregar ranking historico por fecha.
- Mostrar detalle de puntos por partido.
- Agregar exportacion en CSV.
- Agregar filtros por usuario.
- Mejorar presentacion visual de posiciones.

## Ideas futuras

- Notificaciones antes del cierre de pronosticos.
- Invitaciones por link.
- Liga privada con codigo de acceso.
- Multiples torneos o ediciones.
- Panel de estadisticas.
- Comparador de pronosticos entre usuarios.
- Modo solo lectura publico para ranking.
- Backend dedicado si el proyecto crece mas alla de GitHub Pages.
