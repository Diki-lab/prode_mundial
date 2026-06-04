# Dia 06 - Mobile UI

## Estado

COMPLETADO ✅

Validacion local: OK ✅

No se avanza al Dia 7.

## Objetivo

Pulir mobile y legibilidad general sin cambiar la logica ya validada.

## Dependencia estricta

No puede comenzar si `DIA_05_TEST_FLUJO_COMPLETO.md` no esta marcado como `COMPLETADO`.

## Tareas

- Verificar que Dia 5 este `COMPLETADO`.
- Revisar header mobile.
- Revisar tabs y navegacion en pantallas chicas.
- Revisar tablas de fixture, pronosticos, ranking y grupos.
- Ajustar inputs numericos para celulares.
- Revisar login/registro en mobile.
- Revisar panel admin mobile.
- Mantener textos dentro de sus contenedores.

## Archivos permitidos

- `index.html`.
- `docs/AVANCE_PROYECTO.md`.
- `docs/FUNCIONALIDADES_PENDIENTES.md`.
- `docs/roadmap-codex/DIA_06_MOBILE_UI.md`.

## Archivos prohibidos

- `index_backup.html`.
- `index_backup_20260527_1651.html`.
- `prode_mundial_2026_web.html`.
- Cambios funcionales no relacionados con UI.
- Imagenes salvo pedido explicito.

## Criterios de aceptacion

- Login entra en pantallas chicas.
- Header no superpone textos ni botones.
- Tablas son navegables horizontalmente cuando haga falta.
- Botones principales son tocables.
- Panel admin sigue usable en mobile.
- Tema claro/oscuro sigue funcionando.

## Tests obligatorios

- Probar ancho mobile aproximado 360px.
- Probar ancho tablet aproximado 768px.
- Probar desktop.
- Verificar fixture.
- Verificar pronosticos.
- Verificar ranking.
- Verificar admin.
- Verificar login/registro.

## Resultado esperado

La app queda presentable y usable para participantes desde celular.

## Resultado de ejecucion - 4 de junio de 2026

Se ejecuto solamente el Dia 6. No se modifico logica Firebase, ranking,
pronosticos ni roles admin. No se avanzo al Dia 7.

### Cambios realizados

- Se reforzo el header para tablet con areas de grilla separadas para marca,
  usuario y navegacion.
- Se mejoro el header mobile para evitar superposicion de logo, usuario,
  acciones superiores y accesos principales.
- Se mantuvo visible la informacion del usuario en mobile con truncado seguro
  para nombres, email y rol.
- Se cambio la navegacion secundaria mobile a tabs horizontales desplazables,
  evitando que los textos se aprieten en una grilla fija.
- Se aumento el alto minimo y tamano tactil de botones principales.
- Se ajustaron tablas para scroll horizontal y anchos minimos mas razonables en
  fixture, pronosticos, ranking, jugadores y grupos.
- Se mejoraron controles admin en mobile apilandolos en una columna.
- Se ajusto login/registro para pantallas chicas con scroll vertical y titulo
  mas compacto.
- Se agregaron reglas especificas para pantallas de hasta 420px.

### Validaciones ejecutadas

- Dia 5 verificado como `COMPLETADO`.
- Sintaxis del script clasico embebido en `index.html`: OK.
- Balance de llaves CSS: OK.
- Tests logicos embebidos en `runTests()`: OK.
- Revision estatica de reglas para 360px, 768px y desktop: OK.

### Validaciones no ejecutadas en este entorno

- Capturas o prueba visual interactiva real a 360px, 768px y desktop, porque no
  hay navegador interactivo disponible en el entorno de ejecucion.

### Riesgos detectados

- Al ser una mejora CSS sin captura real, puede requerir ajuste fino posterior
  en dispositivos reales, especialmente en nombres o emails largos.
- Las tablas siguen usando scroll horizontal en mobile cuando tienen muchas
  columnas; esto es intencional para no cambiar estructura ni logica.

### Archivos modificados

- `index.html`
- `docs/AVANCE_PROYECTO.md`
- `docs/FUNCIONALIDADES_PENDIENTES.md`
- `docs/roadmap-codex/DIA_06_MOBILE_UI.md`

## Prompt para iniciar el dia

```text
Ejecuta el Dia 6 - Mobile UI. Primero verifica que DIA_05_TEST_FLUJO_COMPLETO.md este COMPLETADO. Si no lo esta, detente. Si esta completo, ajusta solo UI/mobile sin alterar logica funcional validada.
```
