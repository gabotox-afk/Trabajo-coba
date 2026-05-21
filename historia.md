# PAC-MAN: Análisis del Diseño del Juego

## El mapa / Laberinto

El laberinto de Pac-Man es una grilla fija de 28 columnas por 36 filas de tiles. El área jugable efectiva es de 28 por 31 tiles.

### Elementos estructurales del mapa

- **Paredes azules**: bloquean el movimiento. Cambian a blanco brevemente cuando se completa un nivel.
- **Diseño del laberinto**: hay un solo diseño que se repite cada nivel. Solo cambian la velocidad, las frutas y el comportamiento de los fantasmas.
- **Túneles laterales**: en el medio del mapa, a izquierda y derecha, hay dos pasadizos que conectan ambos lados del laberinto. Dentro del túnel, tanto Pac-Man como los fantasmas se ralentizan, pero los fantasmas se ralentizan más que Pac-Man.
- **Ghost House**: caja rectangular en el centro del mapa donde nacen y respawnean los fantasmas. Tiene una abertura en la parte superior.
- **Zona de no-up**: cuatro intersecciones específicas donde los fantasmas, en modo persecución, no pueden girar hacia arriba.

## Pac-Man

### Movimiento

- Se mueve en 4 direcciones: arriba, abajo, izquierda, derecha. No hay diagonales.
- Velocidad base: 80 píxeles por segundo en el nivel 1.
- La velocidad aumenta gradualmente con cada nivel hasta el nivel 5, donde alcanza su máximo.
- **Buffer de input**: si se presiona una dirección antes de llegar a una intersección, Pac-Man registra ese input y gira apenas puede.
- **Cornering**: Pac-Man puede cortar las esquinas tomando la diagonal en una intersección, lo que le da una ventaja sobre los fantasmas.

### Vidas

- Se inicia con 3 vidas (configurable a 1, 2, 3 o 5).
- Se gana una vida extra al alcanzar los 10.000 puntos.

## Pac-Dots

- Hay 240 puntos pequeños distribuidos por el laberinto.
- Cada uno vale 10 puntos.
- Al comer un punto, Pac-Man se ralentiza por 1 frame. Los fantasmas no se ralentizan al pasar por puntos.
- Comer los 240 puntos más las 4 píldoras de poder completa el nivel.

## Power Pellets

- Son los 4 puntos grandes ubicados en las esquinas del mapa.
- Cada una vale 50 puntos.
- Al comerse una, activan el modo **Frightened** en los fantasmas:
  - Los fantasmas se vuelven azul oscuro.
  - Invierten su dirección instantáneamente.
  - Se mueven más lento y de forma aleatoria en cada intersección.
  - Pac-Man puede comerlos.
- Duración del efecto: depende del nivel. En el nivel 1 dura 6 segundos. A partir del nivel 19 las píldoras dejan de funcionar.

### Comer fantasmas asustados

| Fantasma comido | Puntos |
|----------------|--------|
| 1° | 200 |
| 2° | 400 |
| 3° | 800 |
| 4° | 1.600 |

Cuando se come un fantasma, sus ojos vuelven a la Ghost House para regenerarse.

## Los Fantasmas

Cada fantasma tiene una IA propia. Todos siguen el mismo sistema base pero con un tile objetivo diferente.

### Sistema general de IA

Los fantasmas no usan pathfinding. El algoritmo es el siguiente:

1. En cada intersección, miran las direcciones disponibles, excepto retroceder.
2. Calculan cuál los acerca más en línea recta a su tile objetivo.
3. Eligen esa dirección. Si hay empate, la prioridad es: arriba, izquierda, abajo, derecha.

### Modos de comportamiento

1. **Scatter**: cada fantasma va a su esquina del mapa.
2. **Chase**: cada uno persigue a Pac-Man con su lógica.
3. **Frightened**: cuando se come una píldora de poder.
4. **Eaten**: cuando son comidos, sus ojos vuelven a la casa.

El ciclo Scatter/Chase está temporizado. En el nivel 1:

`7s Scatter → 20s Chase → 7s Scatter → 20s Chase → 5s Scatter → 20s Chase → 5s Scatter → Chase permanente`

Cuando cambian de modo, todos invierten su dirección instantáneamente.

### Blinky (Rojo)

- **Esquina de scatter**: arriba-derecha.
- **Objetivo en chase**: el tile exacto donde está Pac-Man.
- **Cruise Elroy**: cuando quedan pocos puntos en el laberinto (alrededor de 20 puntos en el nivel 1), Blinky acelera y se vuelve más rápido que Pac-Man. En niveles avanzados activa este modo antes y dos veces.

### Pinky (Rosa)

- **Esquina de scatter**: arriba-izquierda.
- **Objetivo en chase**: el tile que está 4 casillas adelante de Pac-Man en la dirección a la que mira.
- Su intención es emboscar: no persigue, intenta cortar el paso.

### Inky (Azul)

- **Esquina de scatter**: abajo-derecha.
- **Objetivo en chase**: usa la posición de Blinky y la posición de Pac-Man.
  1. Toma un punto 2 casillas adelante de Pac-Man.
  2. Traza un vector desde Blinky hasta ese punto.
  3. Duplica ese vector. Ese es su objetivo.
- Su comportamiento depende de dónde esté Blinky.

### Clyde (Naranja)

- **Esquina de scatter**: abajo-izquierda.
- **Objetivo en chase**:
  - Si está a más de 8 tiles de Pac-Man, lo persigue como Blinky (apunta al tile de Pac-Man).
  - Si está a menos de 8 tiles, cambia su objetivo a su esquina de scatter y se aleja.

### Velocidades de los fantasmas

- Son ligeramente más lentos que Pac-Man.
- En los túneles laterales, se ralentizan al 40-50%.
- En modo Frightened, se ralentizan al 50%.
- Blinky en modo Elroy es el único que supera la velocidad de Pac-Man.

## Las Frutas

Aparecen 2 veces por nivel en una posición fija debajo de la Ghost House, después de que Pac-Man come 70 puntos, y otra vez después de 170 puntos. Permanecen 10 segundos.

### Tabla de frutas por nivel

| Nivel | Fruta | Puntos |
|-------|-------|--------|
| 1 | Cereza | 100 |
| 2 | Fresa | 300 |
| 3-4 | Naranja | 500 |
| 5-6 | Manzana | 700 |
| 7-8 | Uvas | 1.000 |
| 9-10 | Galaxian | 2.000 |
| 11-12 | Campana | 3.000 |
| 13+ | Llave | 5.000 |

## Objetivo y Progresión

### Objetivo de cada nivel

Comer los 240 puntos más las 4 píldoras de poder del laberinto. Al lograrlo, el laberinto parpadea y se pasa al siguiente nivel.

### Condición de derrota

Si un fantasma toca a Pac-Man en modo Scatter o Chase, se pierde una vida. Al perder todas las vidas, Game Over.

## Sonido

- Música de intro al iniciar.
- **Waka-waka**: sonido continuo mientras Pac-Man come puntos. Son dos tonos alternados rápidamente.
- **Sirena de fondo**: zumbido grave que acelera a medida que se comen puntos.
- Cambio de música cuando los fantasmas están asustados.
- Tintineo agudo cuando los ojos vuelven a la casa.
- Sonido descendente al morir Pac-Man.

## Sistema de Puntuación

| Elemento | Puntos |
|----------|--------|
| Punto pequeño | 10 |
| Píldora de poder | 50 |
| Fantasma 1 | 200 |
| Fantasma 2 | 400 |
| Fantasma 3 | 800 |
| Fantasma 4 | 1.600 |
| Frutas | 100 a 5.000 |
| Vida extra | a los 10.000 |
