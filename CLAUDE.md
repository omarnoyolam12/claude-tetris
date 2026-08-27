# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Qué es

Tetris clásico en JavaScript vanilla con HTML5 Canvas. Sin dependencias, sin `package.json`, sin proceso de build ni transpilación. Tres archivos: `index.html`, `style.css`, `game.js`.

## Ejecutar

```bash
open index.html                 # abrir directamente (macOS)
python3 -m http.server 8000     # o servir estáticamente y abrir http://localhost:8000
```

No hay comandos de build, lint ni test. No existe suite de pruebas: la verificación es manual en el navegador (abrir la página y jugar).

## Arquitectura

Toda la lógica vive en `game.js` (~300 líneas, un solo scope de módulo, sin clases). Puntos clave para no romper invariantes:

- **Tablero**: matriz `ROWS × COLS` (`board`). Cada celda es `0` (vacía) o un índice `1–7` que es a la vez el tipo de pieza y el índice en `COLORS` / `PIECES`. Esos tres arrays van alineados por índice (posición `0` es `null` de relleno).
- **Piezas**: matrices cuadradas en `PIECES`. Rotación = `rotateCW` (transpuesta + reverso de filas). `tryRotate` aplica wall kicks probando desplazamientos `[0, -1, 1, -2, 2]` antes de descartar el giro.
- **Colisión**: `collide(shape, ox, oy)` es la única comprobación de límites y solape; toda mutación de posición/rotación debe pasar por ella antes de aplicarse.
- **Game loop**: `loop(ts)` con `requestAnimationFrame`, acumula `dropAccum` y baja una fila cuando supera `dropInterval`. `dropInterval = max(100, 1000 - (level-1)*90)`, recalculado en `clearLines`.
- **Ciclo de bloqueo**: `lockPiece` → `merge` (fija la pieza en `board`) → `clearLines` → `spawn`. `spawn` mueve `next` a `current`, genera nueva `next`, y si la nueva pieza ya colisiona llama a `endGame`.
- **Puntuación**: `LINE_SCORES` (`[0,100,300,500,800]`) × `level`; hard drop +2/celda, soft drop +1/fila. `level` sube cada 10 líneas.
- **Estado global mutable**: `board, current, next, score, lines, level, paused, gameOver, dropInterval, dropAccum, lastTime, animId`. `init()` los resetea todos y es el handler de reinicio.

El DOM se consulta una sola vez al cargar (constantes `canvas`, `ctx`, `scoreEl`, etc.); `index.html` debe mantener esos `id`. Si cambias `COLS`, `ROWS` o `BLOCK`, ajusta también `width`/`height` del `<canvas id="board">` en `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`).

`style.css` es estética dark/arcade; el `.overlay` sirve tanto para PAUSA como para GAME OVER, alternando con la clase `hidden`.

## Flujo de Git

El proyecto tiene convenciones propias de ramas y commits (ramas `feature/`, `fix/`, etc.; commits con emoji + tipo; flujo develop-first). Usa la skill `git-workflow` o el agente `git-workflow-manager` para crear ramas, commits, push y merges. Los mensajes de commit van en español.
