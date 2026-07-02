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
      "status": "string ('closed' | 'finished', opcional)"
    }
  ],
  "isResultLocked": "boolean (Bloqueo de edición de marcadores oficiales)",
  "arePredictionsLocked": "boolean (Bloqueo manual global de edición de pronósticos)",
  "updatedAt": "string (ISO timestamp del guardado)"
}
```

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
* **Estructura Visual del Bracket (Tournament Bracket):** Rediseñado para mostrarse de izquierda a derecha en un flujo horizontal continuo de columnas que agrupan los partidos por fase (16avos de Final, Octavos, Cuartos, Semifinales, Gran Final).
  * **Contenedor con Scroll Horizontal:** Implementa scroll lateral táctil y en desktop a través de `overflow-x-auto` (`.bracket-scroll-container`) para garantizar la accesibilidad en pantallas móviles sin romper la visual.
  * **Distribución Vertical Dinámica:** Se utiliza Flexbox con `justify-around` y altura automática (sin `height` fijo en el contenedor de columnas), lo cual hace que el fondo gris se estire exactamente al alto real del contenido y las columnas se distribuyan parejas por `align-items:stretch`.
  * **Detección de Fase:** Cada partido se clasifica por fase evaluando su campo `group` contra un set de expresiones regulares (16avos, octavos, cuartos, semifinal, tercer puesto, final). La detección de "Gran Final" excluye explícitamente los grupos que matchean cualquier otra fase, porque el texto "Final" aparece como sufijo en el nombre de todas las fases previas (ej. "Cuartos de Final"); sin esa exclusión, cada partido de las demás fases se duplicaba también en la columna de la Final.
  * **Columnas clickeables:** El título de cada fase es un `<button>` que llama a `scrollBracketToPhase(phaseId)`, haciendo `scrollIntoView` horizontal sobre el `<div id="bracketPhase16|8|4|Semi|Final">` correspondiente dentro de `.bracket-scroll-container`.
  * **Placeholders del cuadro completo:** `renderColumn(phaseId, title, list, slotCount)` rellena con tarjetas "Por definir" (sin inputs) hasta completar el cupo fijo de la fase: 16avos=16, Octavos=8, Cuartos=4, Semifinal=2, Final=1.
  * **Tarjetas de Partidos Compactas e Interactivas:** Cada tarjeta muestra el ID, fecha y hora, banderas emojis oficiales de países (vía `getTeamEmoji`), nombre de los equipos, y mantiene intactas las funciones del monolith:
    * *Admin (Resultados oficiales):* Expone inputs `homeGoals-${id}` y `awayGoals-${id}` (número grande junto al equipo) y botón de "Guardar" llamando a `saveMatchResult`.
    * *Jugadores:* El número grande junto a cada equipo es el **resultado real** (`match.homeGoals`/`match.awayGoals`, solo lectura). La línea inferior "Pronóstico:" muestra el pronóstico del usuario (`prediction.home`/`prediction.away`), editable vía inputs `predictionHome-${id}` / `predictionAway-${id}` llamando a `savePredictionRow` / `editPredictionRow`. El esquema de datos en Firestore no cambió, solo qué valor se muestra en cada posición.
  * **Tercer Puesto:** Se renderiza por separado debajo del bracket para mantener el árbol estético de la Gran Final completamente despejado.
* **Carga manual de cruces por fase:** generalizada en `PHASE_MANUAL_CONFIGS` + `showManualPhaseForm(phaseKey)` / `saveManualPhase(phaseKey)`, reutilizada por los wrappers `showManual16vosForm`/`saveManual16avos` (fecha/hora fija en `SCHEDULE_16AVOS`, ids 73-88) y `showManualOctavosForm`/`saveManualOctavos` (fecha/hora editable por el admin en `SCHEDULE_OCTAVOS`, ids 89-96, porque el calendario oficial de Octavos todavía no está confirmado). Ambos guardan el mismo esquema de partido (`id, date, time, group, home, away, homeGoals, awayGoals, kickoff`) en `prode/mundial2026.matches` vía `window.prodeFirebase.saveData`.
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
