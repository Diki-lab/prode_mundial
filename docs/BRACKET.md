# Memoria del Módulo Eliminatorias (Bracket)

Este archivo documenta en detalle el menú **Eliminatorias** de `index.html`: cómo está
armado, qué decisiones de diseño lo llevaron a su forma actual y qué queda pendiente.
Es un anexo de `docs/memoria.md` — leer ese archivo primero para contexto general del
proyecto (arquitectura, esquema de Firebase, roles).

---

## 1. Estado actual

El bracket muestra 5 columnas (16avos → Octavos → Cuartos → Semifinales → Gran Final) más
una tarjeta de Tercer Puesto aparte, en flujo horizontal con scroll (`.bracket-scroll-container`,
`overflow-x-auto`). Todo se renderiza desde `renderEliminatorias()` sobre
`#bracketScrollContainer`.

**El menú es SOLO LECTURA.** No tiene inputs ni botones de guardado: muestra el resultado
real, el pronóstico del usuario logueado y los puntos ganados, pero todos esos datos se
cargan desde otras pantallas (ver sección 3).

### Orden visual (FIFA), no orden de ID

`BRACKET_DISPLAY_ORDER` fija el orden de arriba hacia abajo de cada columna. No es orden
ascendente de id: los ids consecutivos de cada array son la pareja que efectivamente se
enfrenta / alimenta el mismo cruce de la fase siguiente.

```js
const BRACKET_DISPLAY_ORDER = {
  16:    [74, 77, 73, 75, 83, 84, 81, 82, 76, 78, 79, 80, 86, 88, 85, 87],
  8:     [89, 90, 93, 94, 91, 92, 95, 96],
  4:     [97, 98, 99, 100],
  semi:  [101, 102],
  final: [104],
  third: [103]
};
```

### Topología de avance

`BRACKET_ADVANCEMENT` es la tabla fija (validada contra el cuadro oficial FIFA 2026) de
qué partido anterior alimenta cada slot (`home`/`away`) de cada cruce, y si toma al
ganador o al perdedor del partido de origen:

```js
const BRACKET_ADVANCEMENT = [
  { id: 89,  group: 'Octavos de Final', home: { from: 74, result: 'winner' }, away: { from: 77,  result: 'winner' } },
  { id: 90,  group: 'Octavos de Final', home: { from: 73, result: 'winner' }, away: { from: 75,  result: 'winner' } },
  { id: 91,  group: 'Octavos de Final', home: { from: 76, result: 'winner' }, away: { from: 78,  result: 'winner' } },
  { id: 92,  group: 'Octavos de Final', home: { from: 79, result: 'winner' }, away: { from: 80,  result: 'winner' } },
  { id: 93,  group: 'Octavos de Final', home: { from: 83, result: 'winner' }, away: { from: 84,  result: 'winner' } },
  { id: 94,  group: 'Octavos de Final', home: { from: 81, result: 'winner' }, away: { from: 82,  result: 'winner' } },
  { id: 95,  group: 'Octavos de Final', home: { from: 86, result: 'winner' }, away: { from: 88,  result: 'winner' } },
  { id: 96,  group: 'Octavos de Final', home: { from: 85, result: 'winner' }, away: { from: 87,  result: 'winner' } },
  { id: 97,  group: 'Cuartos de Final', home: { from: 89, result: 'winner' }, away: { from: 90,  result: 'winner' } },
  { id: 98,  group: 'Cuartos de Final', home: { from: 93, result: 'winner' }, away: { from: 94,  result: 'winner' } },
  { id: 99,  group: 'Cuartos de Final', home: { from: 91, result: 'winner' }, away: { from: 92,  result: 'winner' } },
  { id: 100, group: 'Cuartos de Final', home: { from: 95, result: 'winner' }, away: { from: 96,  result: 'winner' } },
  { id: 101, group: 'Semifinales',      home: { from: 97, result: 'winner' }, away: { from: 98,  result: 'winner' } },
  { id: 102, group: 'Semifinales',      home: { from: 99, result: 'winner' }, away: { from: 100, result: 'winner' } },
  { id: 103, group: 'Tercer Puesto',    home: { from: 101, result: 'loser' }, away: { from: 102, result: 'loser' } },
  { id: 104, group: 'Final',            home: { from: 101, result: 'winner' }, away: { from: 102, result: 'winner' } }
];
```

`ADVANCEMENT_BY_ID` es el mismo array indexado por `id` de destino (`Map`), para poder
resolver "¿de dónde viene el equipo del slot 97?" en O(1) sin recorrer el array — se usa
tanto para propagar equipos reales como para mostrar procedencia en placeholders (sección
siguiente). 16avos (ids 73-88) no tiene entrada en `BRACKET_ADVANCEMENT`: viene de la fase
de grupos o de carga manual, no de un cruce anterior.

### Resolución de ganador

- `getKnockoutWinnerSide(match)`: gana por goles a los 120' si no hay empate; si hay
  empate, gana por penales (`homePenalties`/`awayPenalties`). Devuelve `null` si el
  partido no se jugó o está empatado sin penales cargados todavía.
- `computeBracketAdvancement(matches)`: recorre `BRACKET_ADVANCEMENT`, resuelve el
  ganador/perdedor de cada partido origen con `getKnockoutWinnerSide` y completa
  `home`/`away` del cruce siguiente **sin pisar** un cruce que ya tenga equipo cargado
  (manual o de una propagación previa). Devuelve `{ matches, changed }`; si `changed` es
  `true` y el usuario es admin, `renderEliminatorias()` persiste el avance en
  `localStorage` + Firestore (`window.prodeFirebase.saveData`) después de pintar el
  cuadro, envuelto en `try/catch` para que un error de red no rompa el render.

### Placeholders con procedencia

Cada fase se recorre por los ids fijos de `BRACKET_DISPLAY_ORDER` (no por texto de
`group`), así todas las fases muestran siempre su cupo completo de slots, en orden FIFA,
incluso sin datos cargados. Para cada id: si existe un partido real en `matches` (cargado
manualmente o propagado) se usa ese; si no, se arma un objeto de solo-render que **nunca
se persiste**, solo para pintar.

`renderMatchCard(match)` es la única función de render de tarjeta, usada tanto para
partidos resueltos como para slots sin definir. Cuando un lado (`home`/`away`) está
vacío, en vez de un genérico "Por definir" muestra la procedencia real leyendo
`ADVANCEMENT_BY_ID`: `"Ganador #74"`, `"Perdedor #101"` (Tercer Puesto). En cuanto el
partido de origen se resuelve, ese texto se reemplaza por el nombre del equipo real.
`renderPlaceholderCard` (la función vieja, separada, sin procedencia) quedó **jubilada**:
unificar todo en `renderMatchCard` fue también lo que resolvió la falta de altura
uniforme entre tarjetas reales y placeholders (ver sección 4).

Tercer Puesto (id 103) se renderiza siempre, aparte del árbol principal (debajo, en su
propia tarjeta), para no cargar visualmente el camino hacia la Gran Final.

### Alineación vertical

Cada columna usa `justify-around` + `gap-4` (Tailwind) sobre un contenedor de altura
compartida (`items-stretch` en la fila que contiene las 5 columnas). Con esas dos
condiciones, el punto medio de cada cruce queda centrado exactamente entre sus dos
partidos de origen **sin necesidad de multiplicadores de espaciado por fase** — es una
propiedad matemática de `justify-content: space-around` con `gap` fijo: la única
condición real es que **todas las tarjetas midan lo mismo**, en todas las fases (ver
pendiente en sección 5).

### Conectores SVG

Un `<svg id="bracketConnectorsSvg">` absoluto (`position:absolute; inset:0; z-0`) vive
como primer hijo de `#tournamentBracketContainer`, detrás de las columnas (que llevan
`relative z-[1]` explícito). Cada tarjeta se identifica con el atributo
`data-match-slot="{id}"` (**no** `id=`, ver lección en sección 4). Después de pintar el
HTML, `scheduleBracketConnectorsRedraw()` (debounced vía `requestAnimationFrame`) llama a
`renderBracketConnectors()`, que mide con `getBoundingClientRect()` cada tarjeta origen y
destino de `BRACKET_ADVANCEMENT` y traza una polilínea en escuadra (`M`/`H`/`V`/`H`) entre
ambas. Tercer Puesto no tiene conector propio: no vive dentro de
`tournamentBracketContainer`, así que se salta solo (no hay tarjeta que matchee su
`data-match-slot`).

El `<svg>` está **dentro** del mismo contenedor que scrollea horizontalmente junto con las
tarjetas, así que no hace falta escuchar el evento `scroll` para recalcular — se mueve
solo con el contenido. Sí se recalcula en `resize` de la ventana (con el mismo debounce) y
en cada `renderEliminatorias()`.

---

## 2. Esquema de datos (confirmado en código)

Eliminatorias no tiene esquema propio: usa el mismo documento que Fixture.

- **`matches`** (`prode/mundial2026.matches`, array): `{ id, date, time, kickoff, group,
  home, away, homeGoals, awayGoals, homePenalties, awayPenalties, status? }`. Los ids de
  eliminatorias son fijos: 73-88 (16avos), 89-96 (Octavos), 97-100 (Cuartos), 101-102
  (Semis), 103 (Tercer Puesto), 104 (Final). `homePenalties`/`awayPenalties` solo se
  completan cuando el cruce quedó empatado a los 120'.
- **Predicciones** (`prode/mundial2026/predictions/{uid}`): cada pronóstico es
  `{ player, matchId, home, away }`. Sin distinción entre fase de grupos y eliminatorias
  a nivel de esquema; `matchId` es el id fijo de arriba.
- **`status`** (`'closed' | 'finished'`, opcional) es señal de **bloqueo de edición**
  (`isMatchStatusClosed`), no de resultado cargado. La señal real de "hay resultado
  oficial" es `hasOfficialResult(match)` (`homeGoals`/`awayGoals` con valor distinto de
  `''`), usada tanto por el motor de puntos (`pointsFor`) como por
  `getKnockoutWinnerSide` para resolver el ganador del cruce.

---

## 3. Arquitectura de carga (dónde se carga qué, por rol)

| Dato | Dónde se carga | Quién |
|---|---|---|
| Resultado a los 120' (`homeGoals`/`awayGoals`) | Fixture (todas las fases, incluida eliminatorias) | Admin, vía `saveMatchResult(id)` |
| Definición manual de cruces (16avos/Octavos) | Panel Admin → `#jugadores` → tarjeta "Cruces de Eliminatorias" | Admin, vía `PHASE_MANUAL_CONFIGS` / `showManualPhaseForm(phaseKey)` / `saveManualPhase(phaseKey)` |
| Penales en empates a los 120' | Panel Admin → `#jugadores` → tarjeta "Definición por penales" (`#penaltiesPanelCard`) | Admin, vía `saveMatchResult(id)` leyendo `penaltiesHome-{id}`/`penaltiesAway-{id}` |
| Pronóstico del usuario | Pestaña Predicciones (todas las fases) | Cada usuario logueado, sobre sus propios pronósticos |
| Eliminatorias (menú) | — | Nadie: **solo lectura** |

`PHASE_MANUAL_CONFIGS` sigue siendo la config genérica reutilizada por
`showManual16vosForm`/`saveManual16avos` (fecha/hora fija, `SCHEDULE_16AVOS`, ids 73-88) y
`showManualOctavosForm`/`saveManualOctavos` (fecha/hora editable, `SCHEDULE_OCTAVOS`, ids
89-96, porque el calendario oficial de Octavos no estaba confirmado al momento de migrar
esto). Ambos guardan el mismo esquema de partido de siempre
(`id, date, time, group, home, away, homeGoals, awayGoals, kickoff`) en
`prode/mundial2026.matches` vía `window.prodeFirebase.saveData` — se **movió** el HTML/JS
tal cual desde el menú Eliminatorias al panel Admin, sin tocar el esquema de persistencia.

La tarjeta de penales (`#penaltiesPanelCard`) lista los cruces de eliminatorias empatados
a los 120' sin penales cargados y expone `penaltiesHome-{id}`/`penaltiesAway-{id}` +
botón "Guardar", que llama a `saveMatchResult(id)` — la misma función que usa Fixture para
goles, reutilizada para no duplicar lógica de guardado/persistencia.

---

## 4. Lecciones y decisiones clave

### Bug histórico: ids duplicados en el DOM

El bracket llegó a tener sus propios inputs de carga de resultados y pronósticos, con el
mismo patrón de id que usaban Fixture/Predicciones para el mismo partido (ej.
`homeGoals-{id}` en ambos lados). `document.getElementById` agarraba el primer elemento
que encontraba en el DOM, no necesariamente el visible/activo, y en más de una ocasión
terminó **borrando resultados reales** ya cargados. Se parchó en su momento con ids
prefijados (`bracketHomeGoals-{id}`, etc.) y un flag `fromBracket` en
`saveMatchResult`/`savePredictionRow` para leer el input correcto según el origen.

Ese parche se eliminó de raíz: el bracket pasó a ser **solo lectura** (sin inputs, sin
botones de guardado), y las tarjetas se identifican con el atributo `data-match-slot`
(no con `id=`) para los conectores SVG. **Regla permanente: no reintroducir inputs ni ids
con el patrón `bracket*` en el menú Eliminatorias.** Si en el futuro hace falta editar
algo desde ahí, la forma correcta es un control que redirija a Fixture/Admin, no un input
propio con un id que pueda volver a colisionar.

### Diseño unidireccional, no espejado

El bracket se muestra como 5 columnas de izquierda a derecha (16avos → Final), **no**
espejado en V como el cuadro oficial de FIFA (con las dos mitades del cuadro convergiendo
hacia el centro). El espejado se probó y se descartó a propósito: complicaba la detección
de fase y volvió a producir duplicados visuales. El layout unidireccional es más simple de
mantener y es el que se usa en la reimplementación actual.

### Alineación por flex, no por multiplicadores

`justify-around` + `gap` igual en todas las columnas alinea los cruces solo si las
tarjetas miden lo mismo en todas las fases (demostrable: `justify-content: space-around`
con gap fijo tiene la propiedad recursiva de que el centro de un ítem en la fase N es el
promedio de los centros de sus dos ítems fuente en la fase N-1, sea cual sea el valor de
`gap`, siempre que sea el mismo en todas las columnas). No hace falta un gap distinto por
fase (16avos base, Octavos 2x, Cuartos 4x, etc.) — esa idea se evaluó y se descartó por
innecesaria: **la restricción real es la altura uniforme de tarjeta**, ver siguiente
sección.

---

## 5. Pendientes conocidos (honesto, verificado contra código)

### Penales NO se muestran inline: rompen la altura uniforme de tarjeta

En una pasada anterior se dio por verificado (incorrectamente) que los penales ya se
mostraban en la misma línea que el gol. Repasando el código actual:
`getMatchScoreText(match, side)` sí arma el string en un solo `span`
(`"1 (4)"` — gol, luego penales entre paréntesis), y ese `span` está en la misma fila que
el nombre del equipo. El problema es de **layout**, no de markup: ese `span` tiene clase
`w-6 text-right` (ancho fijo de 24px) y **no tiene `whitespace-nowrap`**. Un texto como
`"1 (4)"` no entra en 24px a `text-xs`, así que el navegador lo **envuelve en más de una
línea dentro del mismo span**, lo que visualmente se ve "apilado debajo del gol" aunque
estructuralmente sea un solo elemento en la fila correcta. Esto hace que **solo** las
tarjetas con penales carguen más alto que el resto, rompiendo la premisa de altura
uniforme necesaria para que la alineación por `justify-around` (sección 1/4) funcione
correctamente en esos cruces puntuales.

Fix esperado (no aplicado en esta pasada, que es solo documentación): agregar
`whitespace-nowrap` (y probablemente angostar el font-size o ensanchar el `span`) a
`homeScoreHtml`/`awayScoreHtml` en `renderMatchCard`, para que `"1 (4)"` quede en una sola
línea sin importar el ancho fijo.

### El fondo gris del contenedor no cubre todo el alto del contenido

El contenedor (`bg-slate-50 dark:bg-slate-900/40 rounded-2xl ...` sobre
`#tournamentBracketContainer`) no siempre se estira al alto real del contenido.

**Ambos pendientes probablemente comparten causa raíz:** si una tarjeta con penales crece
más alto que sus vecinas (por el wrap del punto anterior), la columna que la contiene deja
de tener todas sus tarjetas a la misma altura, lo que puede desalinear el cálculo de alto
que usan `items-stretch` / el propio contenido para determinar el alto total de la fila —
y de ahí que el fondo gris quede corto. Antes de tocar el contenedor, conviene resolver
primero el wrap de penales y volver a verificar si el fondo gris se corrige solo.

---

## 6. Dónde seguir

- Antes de tocar `renderEliminatorias`, `BRACKET_ADVANCEMENT`, `BRACKET_DISPLAY_ORDER` o
  `computeBracketAdvancement`: leer este archivo completo, no asumir el estado a partir de
  memoria de conversaciones previas — verificar contra el código actual.
- Antes de tocar el panel admin de cruces/penales: ver sección 3 de este archivo y
  `docs/memoria.md` sección 4 (Panel de Contingencia) para no romper el resto de
  `#jugadores`.
- `docs/FUNCIONALIDADES_PENDIENTES.md` tiene el pendiente de penales/fondo gris registrado
  como bug, con la misma causa raíz que acá.
