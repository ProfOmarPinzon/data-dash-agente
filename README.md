# Data Dash: programa tu agente

En Data Dash un cubo avanza solo y la única decisión es **cuándo saltar**. Tu tarea es programar un **agente**: un programa que, en cada instante, mira el juego y decide `'JUMP'` (saltar) o `'WAIT'` (esperar). El objetivo es que tu agente llegue lo más lejos posible y, si puedes, que complete los niveles y supere al agente de referencia (`RuleAgent`).

Este repositorio contiene el juego ya compilado y **un solo archivo para editar: `agents/StudentAgent.js`**. No necesitas instalar Node.js ni tocar el código del juego.

## 1. Preparar tu copia

1. En GitHub, pulsa **Use this template → Create a new repository**. Así tendrás tu propia copia del repositorio en tu cuenta.
2. Clona tu repositorio en tu computador:

   ```bash
   git clone https://github.com/<tu-usuario>/<tu-repo>.git
   cd <tu-repo>
   ```

3. Inicia un servidor web local en esa carpeta. Basta con cualquiera de estos:

   ```bash
   python3 -m http.server 8080     # viene con Python
   npx serve -l 8080               # si tienes Node.js
   ```

   En Windows, si `python3` no existe, prueba con `py -m http.server 8080`.

4. Abre **http://localhost:8080** en el navegador. Abrir `index.html` con doble clic **no funciona**: el navegador no carga módulos desde `file://`.

## 2. Probar tu agente

1. En el juego, pulsa **Jugar** y elige la modalidad **Agente personalizado**.
2. Elige un nivel y pulsa **Comenzar**. El juego carga `agents/StudentAgent.js` y tu agente juega solo. Tras un choque, empieza otro intento automáticamente.
3. Edita `agents/StudentAgent.js` con tu editor (VS Code, por ejemplo) y **guarda**.
4. Pulsa `Esc` → **Menú principal** → **Jugar** → **Comenzar**. El juego vuelve a cargar el archivo, así que no hace falta recargar la página.

Con la modalidad **Agente basado en reglas** puedes ver cómo juega `RuleAgent` y cambiar sus umbrales. Es la línea base que debes superar: completa el nivel fácil, pero en el intermedio llega al 21 % y en el difícil al 11 %.

> La tecla `H` muestra información técnica: las cajas de colisión y la distancia al siguiente obstáculo. Es muy útil para entender qué ve tu agente.

## 3. El contrato del agente

`agents/StudentAgent.js` exporta por defecto una clase con este formato:

```js
export default class StudentAgent {
  constructor() {
    this.name = 'StudentAgent';            // aparece en los datos (agent_name)
    this.version = '0.1.0';                // cámbiala con cada estrategia nueva (agent_version)
    this.params = { jumpDistance: 0.5 };   // tus parámetros (agent_params)
  }

  reset() {}                               // se llama al comenzar cada intento

  decide(obs) {                            // se llama 120 veces por segundo
    return 'WAIT';                         // debe devolver 'JUMP' o 'WAIT'
  }
}
```

- `decide(obs)` **debe** devolver exactamente `'JUMP'` o `'WAIT'`. Cualquier otro valor, o un error, detiene la partida y el juego muestra el mensaje.
- Devolver `'JUMP'` equivale a **mantener pulsada** la tecla: el cubo salta en cuanto está apoyado. Saltar en el aire no hace nada.
- `reset()` sirve para limpiar la memoria de tu agente entre intentos, si usas memoria.

## 4. La observación (`obs`)

| Campo | Significado |
|---|---|
| `obs.tick`, `obs.time` | Paso de simulación (120 por segundo) y segundos simulados |
| `obs.speed` | Velocidad horizontal del nivel (u/s), constante |
| `obs.progress` | Progreso del intento (0–100 %) |
| `obs.player.x`, `obs.player.y` | Esquina inferior izquierda del cubo (`y = 0` es el suelo) |
| `obs.player.vy` | Velocidad vertical (u/s, positiva hacia arriba) |
| `obs.player.grounded` | `true` si el cubo está apoyado (en el suelo, un bloque o una plataforma) |
| `obs.obstacles` | Hasta **3** obstáculos que aún no has superado, el más cercano primero |
| `obs.next` | El primero de la lista (`null` si ya no quedan) |
| `obs.physics` | `tickRate`, `gravity`, `jumpVelocity`, `playerSize` |

Cada obstáculo de `obs.obstacles` tiene:

| Campo | Significado |
|---|---|
| `type` | `'spike'` (pincho: tocarlo es perder), `'block'` (bloque: se puede aterrizar encima, chocar de frente es perder) o `'platform'` (plataforma elevada: se puede aterrizar encima) |
| `dx` | Distancia horizontal del **frente** del cubo al borde izquierdo del obstáculo. Negativa si el cubo ya está encima |
| `y`, `top` | Altura de la base y de la parte superior del obstáculo |
| `w`, `h` | Ancho y alto |

**Unidades y física útiles.** El cubo mide 1 × 1 u. Un salto desde el suelo sube unas **2,1 u** y dura unos **0,43 s**. Durante un salto el cubo avanza `speed × 0,43` u: unas 3,7 u en el nivel fácil (8,5 u/s), 4,1 u en el intermedio y 4,6 u en el difícil.

## 5. Ideas para mejorar tu agente

1. **Ajusta la distancia de salto** según el tipo de obstáculo (`next.type`). Los pinchos y los bloques no piden lo mismo.
2. **Mira más allá del siguiente obstáculo.** En `obs.obstacles` hay tres. Si van seguidos (pinchos dobles o triples, escaleras), quizá haya que saltar antes o después.
3. **Ten en cuenta la altura.** Si vas sobre un bloque (`player.y = 1`), un obstáculo en el suelo ya no te afecta.
4. **Predice la trayectoria** con `obs.physics`: ¿dónde aterrizarás si saltas ahora? ¿Es un lugar seguro?
5. **Aprende de los datos.** Juega tú mismo, descarga los datos y estudia a qué `next_obstacle_dx` saltaste en los intentos que salieron bien.

Cambia `this.version` cada vez que pruebes una estrategia distinta: así podrás comparar tus versiones en los datos.

## 6. Evaluar tu agente en muchos niveles: el laboratorio

Ver jugar a tu agente nivel a nivel es lento. En la pantalla de inicio, **Laboratorio de agentes** lo hace jugar sin gráficos en decenas o cientos de niveles, en segundos:

1. Elige **Agente personalizado** (o el de reglas, para comparar).
2. Marca los niveles predefinidos que quieras y cuántos **niveles procedurales** (semillas consecutivas desde la primera).
3. Pulsa **Ejecutar**. Verás una fila por nivel y un resumen. La tabla **Comparación de experimentos** reúne todos los que ejecutes en la pestaña.

Cada experimento tiene un `experiment_id` que queda en los datos. **Descargar este experimento** entrega solo sus filas.

- **Compara siempre en las mismas semillas.** Si tu agente y `RuleAgent` juegan niveles distintos, la comparación no vale.
- **Separa semillas de desarrollo y de evaluación.** Ajusta tu agente con las semillas 1–50 y mide la versión final una sola vez en las 1001–1100. Así sabrás si funciona en niveles que no ha visto.
- Si tu agente entra en un bucle infinito, el laboratorio lo detiene a los 5 s y te avisa; la página no se congela.

Más detalles en [experimentos](docs/experimentos.md) y en la [práctica de IA](docs/practica-ia.md).

## 7. Analizar los resultados

Los intentos de tu agente se guardan como los de cualquier jugador. Con **Descargar datos** obtienes un ZIP con `sessions.csv`, `attempts.csv` y `events.csv`. En `sessions.csv`, las columnas `mode = student`, `agent_name`, `agent_version` y `agent_params` identifican qué agente jugó y con qué parámetros.

- [Guía del dataset](docs/guia-datos.md): qué significa cada tabla y cómo analizarla.
- [Diccionario de datos](docs/diccionario-datos.md): cada columna en detalle.

Usa **Borrar datos** antes de una prueba si quieres un dataset limpio.

> Si tu agente no usa azar ni memoria, repetirá **exactamente** el mismo intento cada vez. Para medirlo bien, usa el laboratorio con muchos niveles procedurales (semillas distintas) en lugar de contar muchos intentos idénticos.

## 8. Reglas

- Modifica solo `agents/StudentAgent.js`.
- No uses `Math.random()` si quieres resultados reproducibles. Si lo usas, dilo en tu informe.
- `decide()` se ejecuta 120 veces por segundo: evita bucles largos. Un bucle infinito congela la pestaña (ciérrala y vuelve a abrirla).
- El agente solo puede usar la información de `obs`.

## 9. Entregar tu agente

1. Haz commit y push de tus cambios:

   ```bash
   git add agents/StudentAgent.js
   git commit -m "Agente v0.2.0: salto según el tipo de obstáculo"
   git push
   ```

2. Publica tu juego con tu agente: en tu repositorio, **Settings → Pages → Build and deployment → Deploy from a branch → `main` / `(root)` → Save**. En uno o dos minutos estará en `https://<tu-usuario>.github.io/<tu-repo>/`.
3. Entrega el enlace del repositorio y el de la página, como indique tu docente.

## 10. Problemas frecuentes

| Problema | Solución |
|---|---|
| «No se pudo cargar agents/StudentAgent.js» | Hay un error de sintaxis en tu archivo, o abriste el juego con doble clic. Abre la consola del navegador (`F12` → Consola) para ver la línea del error y usa un servidor local |
| «StudentAgent.decide() devolvió …» | `decide()` debe devolver exactamente `'JUMP'` o `'WAIT'` (en mayúsculas y entre comillas), en todos los caminos del código |
| «… no tiene export default» | El archivo debe contener `export default class StudentAgent { … }` |
| Mis cambios no se ven | Guarda el archivo y vuelve a pulsar **Comenzar**. Si no basta, recarga la página con `Ctrl + Shift + R` (`Cmd + Shift + R` en Mac) |
| La página se congela | Probablemente hay un bucle infinito en `decide()` durante una partida en pantalla. Cierra la pestaña, corrige el código y vuelve a abrir el juego. En el laboratorio, el bucle se detiene solo a los 5 s |
| `console.log` llena la consola | `decide()` se llama 120 veces por segundo: registra solo cuando algo cambie (por ejemplo, cuando decidas saltar) |
