# ANTIGRAVITY.md — Asteroids

Clon del arcade Asteroids en HTML5 Canvas puro. Sin dependencias, sin bundler, sin build.

## Tech Stack
- **HTML5 Canvas 2D** — renderizado vectorial (solo `stroke`/`fill`, blanco sobre negro).
- **JavaScript ES6+ vanilla** — `'use strict'`, clases ES6, script clásico (no ES modules).
- **CSS inline** en `index.html` (centrado flex, fondo `#000`).
- Sin `package.json`, linter, formatter ni tests.

## Arquitectura
```
index.html   # <canvas id="canvas" 800x600> + <script src="game.js">
game.js      # Toda la lógica del juego (~420 líneas)
favicon.svg
README.md
```

`game.js` está organizado en secciones separadas por banners `// ── Nombre ───…`, en este orden:
1. **Input** — `keys` (mantenidas) y `justPressed` + `pressed(code)` (flanco, se consume al leer). Usa `e.code` (`ArrowLeft`, `Space`, …).
2. **Utils** — `wrap`, `dist`, `rand`, `randInt`.
3. **Entidades** (clases) — `Bullet`, `Asteroid`, `PowerUp`, `Ship`, `Particle`. Contrato común:
   - Propiedades `x`, `y`, `vx`, `vy`, `radius`, `dead`.
   - `update(dt)` (dt en segundos) y `draw()` (usa el `ctx` global).
   - Los objetos con vida limitada usan `ttl` y marcan `dead = true`.
4. **Estado del juego** — globals mutables `ship, bullets, asteroids, particles, powerUps, score, lives, level, state, deadTimer, tripleShotTimer, asteroidsKilled, powerUpDropAt, powerUpSpawned`. Funciones `initGame`, `nextLevel`, `spawnAsteroids`, `explode`, `killShip`.
5. **Update** — máquina de estados `state`: `'playing' | 'dead' | 'gameover'`. Colisiones circulares por `dist()`.
6. **Draw** — `draw()`, `drawHUD()`, `drawOverlay()`, `drawLifeIcon()`.
7. **Loop principal** — `requestAnimationFrame`, `dt` clamp a `0.05` s.

Patrones clave:
- **Espacio toroidal**: toda posición de entidad persistente pasa por `wrap(v, W|H)`.
- **Eliminación diferida**: marcar `dead = true` y luego `arr = arr.filter(e => !e.dead)`; nunca hacer `splice` durante la iteración.
- **Tablas por tamaño de asteroide** indexadas 1..3 (índice 0 sin uso): `RADII`, `SPEEDS`, `POINTS`.
- Dimensiones lógicas fijas `W = 800`, `H = 600` (deben coincidir con el `<canvas>`).

## Convenciones
- Indentación 2 espacios, comillas simples, punto y coma, `const`/`let` (nunca `var`).
- Clases en `PascalCase`; funciones/variables en `camelCase`; constantes de tuning en `UPPER_SNAKE` (locales al método o a nivel de módulo).
- Alinear asignaciones consecutivas con espacios (estilo existente).
- Comentarios y texto de UI en **español** (`NIVEL`, `PUNTAJE`, `ESPACIO PARA REINICIAR`).
- Física en unidades por segundo (px/s, rad/s); multiplicar siempre por `dt`.
- Estética vectorial monocromática: `strokeStyle '#fff'`, `lineWidth ~1.5`, `lineJoin 'round'`, fuente `monospace`.
- Al añadir entidades nuevas: seguir el contrato `update(dt)/draw()/dead`, añadir su array al estado, a `initGame`/`nextLevel` y a los bucles de `update()`/`draw()`.
- Mantener el proyecto sin dependencias ni pasos de build salvo que el usuario lo pida.

## Comandos Clave
```bash
# Opción 1: abrir index.html directamente en el navegador
# Opción 2: servidor estático local
npx serve .          # http://localhost:3000
```
No hay build, lint ni tests. Verificar manualmente en el navegador.

## Notas
- **Power-up Disparo Triple** implementado: aparece garantizado una vez por nivel, en la destrucción N sorteada por `resetPowerUpDrop()` (`POWERUP_DROP_MIN..MAX`, acotado a las destrucciones posibles del nivel). Se recoge tocándolo, caduca a los `POWERUP_TTL` (5 s) y da `TRIPLE_DURATION` (10 s) de abanico de 3 balas. Morir lo cancela; cambiar de nivel no.
- La "estrella fugaz" que menciona el README **no está implementada**.
- Demo publicada: https://klerith.github.io/claude-asteroids/
