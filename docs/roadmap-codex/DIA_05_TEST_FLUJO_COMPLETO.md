# Dia 05 - Test Flujo Completo

## Estado

COMPLETADO ✅

Validacion navegador: OK ✅

No se avanza al Dia 6.

## Objetivo

Probar el flujo completo de punta a punta antes de pulir UI: usuario comun, admin, Firebase, bloqueos, pronosticos, resultados y ranking.

## Dependencia estricta

No puede comenzar si `DIA_04_RANKING_PRONOSTICOS.md` no esta marcado como `COMPLETADO`.

## Tareas

- Verificar que Dia 4 este `COMPLETADO`.
- Crear checklist de prueba manual.
- Probar registro de usuario nuevo.
- Probar login/logout.
- Probar guardado de perfil y avatar.
- Probar guardado de pronosticos.
- Probar bloqueo de pronosticos.
- Probar carga de resultados admin.
- Probar ranking y tabla de grupos.
- Documentar fallos encontrados y corregir solo los bloqueantes.

## Archivos permitidos

- `index.html`, solo para corregir bugs bloqueantes.
- `docs/AVANCE_PROYECTO.md`.
- `docs/FUNCIONALIDADES_PENDIENTES.md`.
- `docs/roadmap-codex/DIA_05_TEST_FLUJO_COMPLETO.md`.

## Archivos prohibidos

- `index_backup.html`.
- `index_backup_20260527_1651.html`.
- `prode_mundial_2026_web.html`.
- Cambios grandes de diseno.
- Cambios de arquitectura.

## Criterios de aceptacion

- Flujo jugador completo funciona.
- Flujo admin completo funciona.
- No hay errores nuevos en consola.
- Datos persisten despues de recargar.
- Usuario comun no puede operar admin.
- Ranking refleja resultados cargados.

## Tests obligatorios

- Registro usuario nuevo.
- Login usuario existente.
- Logout.
- Guardar perfil.
- Guardar avatar si Storage esta disponible.
- Guardar pronostico.
- Bloquear pronosticos.
- Intentar guardar con bloqueo activo.
- Login admin.
- Cargar resultado.
- Verificar ranking.
- Verificar tabla de grupos.

## Resultado esperado

La app queda funcionalmente validada antes de invertir tiempo en pulido visual.

## Checklist de prueba manual

- Registro de usuario nuevo.
- Login de usuario existente.
- Logout.
- Guardar perfil.
- Guardar avatar si Storage esta disponible.
- Guardar pronostico.
- Bloquear pronosticos.
- Intentar guardar con bloqueo activo.
- Login admin.
- Cargar resultado.
- Verificar ranking.
- Verificar tabla de grupos.

## Resultado de ejecucion - 4 de junio de 2026

Se ejecuto solamente el Dia 5. No se implementaron funcionalidades nuevas, no se
modifico diseno y no se avanzo al Dia 6.

### Validaciones realizadas

- Dia 4 verificado como `COMPLETADO`.
- Sintaxis del script clasico embebido en `index.html`: OK.
- Tests logicos embebidos en `runTests()`: OK.
- Registro real en Firebase Auth con dos usuarios comunes temporales: OK.
- Login real de ambos usuarios comunes temporales: OK.
- Escritura real de perfil en `users/{uid}` para ambos usuarios: OK.
- Lectura real de `prode/mundial2026` con usuario autenticado: OK.
- Escritura real de pronosticos separados en
  `prode/mundial2026/predictions/{uid}` para ambos usuarios: OK.
- Lectura real de la subcoleccion `predictions` como fuente de ranking: OK.
- Verificacion de que ambos documentos de pronosticos eran visibles para ranking:
  OK.
- Intento de escritura admin sobre `prode/mundial2026` con usuario comun:
  rechazado por Firestore con HTTP 403, OK.
- Limpieza de documentos de pronosticos temporales creados para la prueba: OK.

Usuarios temporales usados:

```text
codex-dia5-a-1780596617541@prode.local
codex-dia5-b-1780596617541@prode.local
```

### Validacion manual en navegador

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

### Fallos encontrados

No se encontraron bugs bloqueantes en las validaciones ejecutadas. No se
modifico `index.html`.

### Estado final

- Dia 5: `COMPLETADO ✅`.
- Validacion navegador: `OK ✅`.
- No se avanza al Dia 6.

## Prompt para iniciar el dia

```text
Ejecuta el Dia 5 - Test Flujo Completo. Primero verifica que DIA_04_RANKING_PRONOSTICOS.md este COMPLETADO. Si no lo esta, detente. Si esta completo, prueba punta a punta y corrige solo bugs bloqueantes.
```
