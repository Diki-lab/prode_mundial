# Memoria Técnica del Proyecto - Prode Mundial 2026

Este archivo sirve como referencia rápida de la arquitectura, base de datos y lógica del Prode para evitar tener que inspeccionar el archivo completo de 5000 líneas (`index.html`).

---

## 1. Arquitectura General
El proyecto está implementado como una **Single Page Application (SPA) estática** hospedada en GitHub Pages.
* **Frontend:** HTML5, CSS vanilla y Javascript ES6 puro sin frameworks ni pasos de build.
* **Persistencia:**
  * **Local:** `localStorage` como fallback de datos y estado.
  * **Remota:** Firebase v12 (Auth & Firestore) consumido a través de CDNs oficiales.
* **Control de Acceso:** El admin oficial es `dmcfarlane@prode.local`. Los demás usuarios registrados adquieren el rol `user`.

---

## 2. Esquema de Datos de Firebase

### A. Colección de Usuarios
* **Ruta:** `/users/{uid}`
* **Descripción:** Almacena perfiles de usuario, roles y preferencias estéticas.
* **Esquema:**
```json
{
  "uid": "string (Firebase Auth UID)",
  "displayName": "string (Nombre de perfil)",
  "email": "string (Email del usuario)",
  "role": "string ('admin' | 'user')",
  "isAdmin": "boolean",
  "providers": ["string (proveedores de autenticación, ej. 'password')"],
  "avatarType": "string ('initial' | 'emoji' | 'color')",
  "avatarValue": "string (Iniciales o emoji)",
  "avatarColor": "string (Código hexadecimal del fondo)",
  "photoURL": "string (URL de Firebase Storage del avatar)",
  "createdAt": "Timestamp (Fecha de creación del perfil)",
  "updatedAt": "Timestamp (Última actualización de perfil)",
  "lastLoginAt": "Timestamp (Último login)"
}
```

### B. Documento Global de Torneo
* **Ruta:** `/prode/mundial2026`
* **Descripción:** Contiene la configuración global del mundial, fixture y bloqueos administrados.
* **Esquema:**
```json
{
  "players": ["string (nombres de jugadores registrados)"],
  "matches": [
    {
      "id": "number (1 al 72)",
      "date": "string (DD/MM/YYYY)",
      "time": "string (HH:MM)",
      "kickoff": "string (ISO 8601 con offset, ej: '2026-06-11T16:00:00-03:00')",
      "group": "string (ej: 'Fecha 1 - Grupo A')",
      "home": "string (Equipo Local)",
      "away": "string (Equipo Visitante)",
      "homeGoals": "number | string (Goles reales, '' si no inició)",
      "awayGoals": "number | string (Goles reales, '' si no inició)",
      "homePenalties": "number | string (Penales local, solo eliminatorias con empate a los 120')",
      "awayPenalties": "number | string (Penales visitante, solo eliminatorias con empate a los 120')",
      "status": "string ('closed' | 'finished', opcional)"
    }
  ],
  "isResultLocked": "boolean (Bloqueo de edición de marcadores oficiales)",
  "arePredictionsLocked": "boolean (Bloqueo manual global de edición de pronósticos)",
  "updatedAt": "string (ISO timestamp del guardado)"
}
```

`status` es solo una señal de bloqueo de edición (ver sección 4), no indica si el resultado está cargado. La señal real de "resultado oficial cargado" es `hasOfficialResult(match)` (`homeGoals`/`awayGoals` con valor, ver `docs/BRACKET.md`), usada tanto para el motor de puntuación como para resolver ganadores de eliminatorias.

### C. Colección de Predicciones por Usuario
* **Ruta:** `/prode/mundial2026/predictions/{uid}`
* **Descripción:** Almacena los pronósticos cargados por cada jugador autenticado.
* **Esquema:**
```json
{
  "uid": "string (UID del usuario)",
  "displayName": "string (Nombre para visualización)",
  "email": "string (Email del usuario)",
  "predictions": [
    {
      "player": "string (Nombre de perfil)",
      "matchId": "number",
      "home": "number (Goles pronosticados local)",
      "away": "number (Goles pronosticados visitante)"
    }
  ],
  "createdAt": "Timestamp",
  "updatedAt": "Timestamp"
}
```

---

## 3. Lógica de Fases y Bracket

### Fase de Grupos
* Consta de 72 partidos cargados estáticamente a través de `buildDefaultMatches()`.
* Los partidos se agrupan en base a la propiedad `group` (ej. `Grupo A` al `Grupo L`).

### Eliminatorias (Knockouts)
* Las llaves de eliminatorias (16avos, octavos, cuartos, semifinales, tercer puesto y final) **se leen directamente** del array `matches` de Firestore.
* El menú Eliminatorias es **SOLO LECTURA** (sin inputs ni botones de guardado): muestra resultado real, pronóstico del usuario y puntos, pero toda la carga de datos (cruces, resultados, penales) se hace desde el panel Admin/Fixture. Ver **`docs/BRACKET.md`** para el detalle completo de topología, esquema visual, arquitectura de carga por rol y las lecciones de diseño que llevaron a este modelo (bug histórico de ids duplicados en el DOM incluido).
* **Generación automática de 16avos de Final:** El administrador dispone de un botón para generar la fase de 16avos de Final en Firestore basándose en los standings de los 12 grupos de la fase de grupos, siguiendo el formato de la Copa Mundial FIFA 2026:
  * `getQualifiedTeams()`: Obtiene 1ros y 2dos de cada grupo (A al L), y ordena los 3ros por Puntos, Diferencia de Goles y Goles a Favor para obtener a los 8 mejores terceros clasificados.
  * `matchThirdsAndLeftover(winners, thirdsList)`: Algoritmo de backtracking recursivo que empareja a los 8 mejores terceros contra 8 de los 9 líderes de grupo (`A, B, D, E, G, I, K, L` y `C`, priorizando dejar a `C` como sobrante), asegurando que ningún tercer puesto juegue contra el ganador de su grupo de origen. Identifica también al único ganador de grupo sobrante (normalmente `C`).
  * `generateRoundOf32()`: Consolida los 7 cruces fijos, el cruce de ganador sobrante contra `2I` y los 8 cruces con mejores terceros. Asigna a cada ID (73 al 88) su fecha y hora oficial exacta desde un mapeo estático (`scheduleMap`), construyendo el campo `kickoff` unificado en formato `YYYY-MM-DDTHH:MM:00-03:00` para mantener consistencia. Los partidos se persisten en Firestore y actualizan reactivamente la vista de llaves y fixture al resolverse la promesa.
* En `renderFixture()`, los partidos se agrupan e iteran dinámicamente por fase según su atributo `group` usando expresiones regulares.
* El método `getPredictionStageName(match)` analiza los campos de partido para formatear el nombre de fases avanzadas (ej. `16avos`, `8vos`, `4tos`, `Semifinal`, `3° Puesto`, `Final`).

---

## 4. Restricciones de Negocio y Tiempos Límite

### Bloqueo de Resultados (Marcadores Oficiales)
* El admin es bloqueado para cargar marcadores si `isResultLocked` es `true`. Evaluado en `updateResult()` mediante la función `isResultEditingBlocked()`.

### Bloqueo de Pronósticos del Jugador
* La edición de predicciones de un partido se bloquea si se cumple cualquiera de las siguientes condiciones en `isPredictionEditableForMatch(match, now)`:
  1. `arePredictionsLocked === true` (Bloqueo manual general).
  2. `isMatchStatusClosed(match) === true` (Estado de partido es `closed` o `finished`).
  3. `hasMatchStarted(match, now) === true` (El partido ya comenzó).
* **Cálculo de fecha/hora de inicio:**
  1. Usa `match.kickoff` (ISO string).
  2. Fallback: Construye un objeto Date en zona horaria de Argentina (`-03:00`) concatenando `match.date` y `match.time` vía `buildArgentinaDateTime()`.
  3. Fallback final: Asume las `00:00` del día del partido (`match.date`).

### Panel de Contingencia y Carga Retroactiva (Sobreescritura Remota)
* **Acceso y Visibilidad:** Protegido bajo la condición estricta de `isAdmin === true` en la UI y reservado en base de datos para el correo `dmcfarlane@prode.local`. Las vistas y componentes tienen la clase `.admin-only` para control visual estricto.
* **Bypass de Restricciones:** Permite omitir las restricciones de tiempo y estado (`hasMatchStarted`, `arePredictionsLocked`, etc.) para realizar la carga manual retroactiva de pronósticos de cualquier usuario registrado.
* **Lógica de Guardado:** Al guardar, realiza la mezcla y persistencia de las predicciones modificadas con las ya existentes para el usuario seleccionado en `/prode/mundial2026/predictions/{uid}`, disparando un refresco visual inmediato de todo el ranking.
* **Seguridad Firestore:** Las reglas en `firestore.rules` permiten la lectura si se está autenticado, y la escritura (`create`, `update`, `delete`) si el `request.auth.uid` coincide con el `{uid}` del documento o bien si el email del usuario autenticado es `dmcfarlane@prode.local`.

---

## 5. Motor de Puntuación (`pointsFor`)

Cada predicción de partido calcula su puntuación en tiempo real basándose en el resultado real de los **120 minutos (tiempo reglamentario + alargue), excluyendo la definición por penales**:

| Condición | Puntos Asignados | Explicación |
| :--- | :--- | :--- |
| `match.homeGoals === ''` o `match.awayGoals === ''` | **0** | Partido aún sin resultado oficial. |
| `pred.home === realHome && pred.away === realAway` | **3** | Acierto exacto del marcador (a los 120'). |
| `Math.sign(pred.home - pred.away) === Math.sign(realHome - realAway)` | **1** | Acierto del ganador/empate (marcador incorrecto a los 120'). |
| Otro caso | **0** | Pronóstico errado. |

### Sanciones Especiales (Leandro Sanction)
* Los pronósticos del jugador "Leandro Iriarte" (o "Leandro") correspondientes a partidos con fecha de kickoff igual o anterior a `2026-06-18T19:00:00-03:00` son omitidos en el ranking general y el podio de inicio mediante `applyPredictionSanctions()`.

---

## 6. Archivos Críticos del Proyecto
* **`/index.html`**: Contiene toda la lógica de visualización (fixture, ranking, posiciones), guardado de predicciones, control de bloqueos, autenticación y la declaración de la API de Firebase.
* **`/firestore.rules`**: Contiene las reglas de seguridad de Firestore que garantizan la integridad de la base de datos (ej. restringir que usuarios comunes editen predicciones ajenas o datos del fixture global).
* **`docs/BRACKET.md`**: Memoria dedicada al módulo de eliminatorias (topología, esquema visual, arquitectura de carga por rol, lecciones de diseño y pendientes conocidos). Consultar antes de tocar `renderEliminatorias`, `BRACKET_ADVANCEMENT` o el panel admin de cruces/penales.
