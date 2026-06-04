# Dia 05 - Test Flujo Completo

## Estado

PENDIENTE.

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

## Prompt para iniciar el dia

```text
Ejecuta el Dia 5 - Test Flujo Completo. Primero verifica que DIA_04_RANKING_PRONOSTICOS.md este COMPLETADO. Si no lo esta, detente. Si esta completo, prueba punta a punta y corrige solo bugs bloqueantes.
```
