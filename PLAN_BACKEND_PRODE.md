# Plan para evolucionar el Prode Mundial 2026 a aplicación full-stack

## 1) Objetivo
Transformar el HTML estático actual (con estado en `localStorage`) en una aplicación web con:
- Backend persistente para pronósticos, partidos y resultados.
- Manejo de usuarios con autenticación y autorización por roles.
- Ranking y tablas calculadas de forma consistente en servidor.
- Base técnica para escalar a múltiples torneos/ediciones.

---

## 2) Alcance funcional (MVP)

### 2.1 Usuarios y seguridad
- Registro / login / logout.
- Recuperación de contraseña por email (fase 2 si se quiere simplificar MVP).
- Roles:
  - `admin`: carga fixture y resultados, bloquea jornada, administra jugadores.
  - `jugador`: carga/edita sus pronósticos dentro de ventana permitida.
- Sesión segura con JWT (cookie httpOnly) o sesión server-side.

### 2.2 Prode
- Fixture de partidos por grupo/fase.
- Carga de resultados reales por admin.
- Carga/edición de pronósticos por usuario.
- Reglas de puntaje configurables (ej. exacto=3, signo=1).
- Ranking global y por fecha.
- Tabla de grupos y cruces de eliminatorias calculados desde resultados.

### 2.3 Operación
- Panel admin básico.
- Auditoría mínima de cambios críticos (resultado modificado por quién/cuándo).
- Seed inicial de Mundial 2026 (equipos, grupos, partidos).

---

## 3) Arquitectura recomendada

## Opción sugerida (rápida y mantenible)
- **Frontend**: React + Vite + TypeScript (migración progresiva desde el HTML actual).
- **Backend API**: Node.js + NestJS (o Express + Zod/TypeScript si prefieren más simple).
- **Base de datos**: PostgreSQL.
- **ORM**: Prisma.
- **Auth**: JWT + refresh token en cookie segura.
- **Infra**: Docker Compose para local (`frontend`, `api`, `db`).

> Alternativa más minimal: mantener frontend vanilla JS y agregar solo API REST + fetch. Ideal si quieren baja inversión inicial.

---

## 4) Modelo de datos (propuesto)

### Tablas principales
- `users`
  - id, name, email (unique), password_hash, role, created_at, updated_at
- `tournaments`
  - id, name, year, status
- `teams`
  - id, name, fifa_code, group_code
- `matches`
  - id, tournament_id, stage, group_code, match_number, kickoff_at, home_team_id, away_team_id, home_goals, away_goals, locked
- `predictions`
  - id, user_id, match_id, pred_home_goals, pred_away_goals, points_awarded, created_at, updated_at
  - unique(user_id, match_id)
- `scoring_rules`
  - id, tournament_id, exact_points, outcome_points, bonus_points (opcional)
- `audit_logs`
  - id, actor_user_id, action, entity, entity_id, payload, created_at

### Reglas de negocio clave
- Si partido `locked=true`, no se puede editar resultado ni pronósticos (salvo admin con override).
- `points_awarded` se recalcula cuando hay cambio de resultado real o de reglas.
- Endpoint de ranking siempre toma datos persistidos y/o vista materializada.

---

## 5) API REST (MVP)

### Auth
- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/logout`
- `GET /auth/me`

### Catálogo
- `GET /teams`
- `GET /tournaments/:id/matches`
- `GET /tournaments/:id/standings`
- `GET /tournaments/:id/bracket`

### Pronósticos
- `GET /predictions/me?tournamentId=...`
- `PUT /predictions/:matchId` (upsert del pronóstico del usuario)

### Admin
- `PUT /admin/matches/:id/result`
- `PUT /admin/matches/:id/lock`
- `PUT /admin/tournaments/:id/rules`
- `GET /admin/audit-logs`

### Ranking
- `GET /rankings?tournamentId=...`
- `GET /rankings?tournamentId=...&groupBy=fecha`

---

## 6) Migración desde el HTML actual

1. **Congelar lógica existente**: extraer JS actual a módulos (`scoring`, `standings`, `bracket`) para reutilizar reglas.
2. **Crear API mock** con contratos estables (OpenAPI).
3. **Reemplazar localStorage** por repositorio de datos remoto:
   - primero modo híbrido (leer API, fallback local).
   - luego full API.
4. **Introducir login** y asociar pronóstico a usuario autenticado.
5. **Panel admin** para resultados y bloqueos.
6. **Eliminar estado crítico del cliente** una vez validada paridad de cálculos.

---

## 7) Roadmap por fases (8-10 semanas)

### Fase 0 (Semana 1): Base técnica
- Monorepo (o repos separados), linters, formato, CI.
- Docker compose y Postgres.
- Esquema inicial Prisma + migraciones.

### Fase 1 (Semanas 2-3): Auth + fixture
- Registro/login/logout + roles.
- Endpoints de equipos/partidos.
- Front consumiendo fixture real.

### Fase 2 (Semanas 4-5): Pronósticos
- Upsert pronóstico por partido y usuario.
- Restricciones por lock/horario.
- UI de “mis pronósticos”.

### Fase 3 (Semanas 6-7): Resultados + ranking
- Carga de resultados por admin.
- Cálculo de puntos y ranking server-side.
- Tabla de grupos y eliminatorias desde backend.

### Fase 4 (Semanas 8-9): Hardening
- Auditoría, validaciones, rate limit, logs.
- Tests integrales + performance básica.
- Backups y monitoreo mínimo.

### Fase 5 (Semana 10): Go-live
- Deploy (API + DB + frontend).
- Migración final de datos de prueba.
- Checklist de operación.

---

## 8) Testing y calidad
- **Unit tests**: scoring, standings, bracket.
- **Integration tests**: API auth/predictions/results/ranking.
- **E2E**: flujo jugador y flujo admin.
- **Seguridad**: OWASP básico (auth, inyección, XSS/CSRF según estrategia de sesión).

---

## 9) Riesgos y mitigaciones
- **Inconsistencia de reglas** entre cliente y servidor → única fuente de verdad en backend.
- **Cambios de fixture** de último momento → modelo editable + seeds versionados.
- **Concurrencia al cerrar jornada** → transacciones DB + locks lógicos.
- **Escalabilidad de ranking** → índices correctos + cálculo incremental/caché.

---

## 10) Próximos pasos concretos (esta semana)
1. Definir stack final (Nest vs Express, React vs mantener vanilla).
2. Aprobar esquema de datos y reglas de puntaje.
3. Crear repositorio base + Docker + Prisma migrations.
4. Implementar Auth y endpoint `GET /matches`.
5. Conectar pantalla fixture al backend.

---

## 11) Entregables esperados del MVP
- API documentada (OpenAPI/Swagger).
- Front con login, fixture, pronósticos, ranking.
- Panel admin para resultados y bloqueos.
- BD persistente con migraciones reproducibles.
- Pipeline CI con tests mínimos obligatorios.
