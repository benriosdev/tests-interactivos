# Tests interactivos DAW y DAM

Tests interactivos para repasar las asignaturas de los ciclos de Desarrollo de Aplicaciones Web y Multiplataforma, con preguntas revisadas y explicadas.

**[Probar la web →](https://benrios.dev/tests/)**

![Portada con el mapa de constelaciones](docs/capturas/portada.gif)

## Por qué existe

Mientras estudiaba DAW y DAM, los tests que encontraba estaban desactualizados, incompletos o tenían preguntas que se acertaban eligiendo la opción más larga. Así que decidí hacer los míos: preguntas propias y revisadas, con una explicación en cada una y ordenadas por asignatura y tema.

También es mi proyecto de aprendizaje. Me sirve para llevar lo que se ve en el ciclo a un producto real y publicado: diseño de interfaz, TypeScript, pruebas automatizadas, accesibilidad, rendimiento y SEO. Con el SEO aprendí a pensar qué debe encontrar un buscador y qué no: sitemap, `noindex`, canonical, datos estructurados y metadatos para compartir. Las decisiones importantes están documentadas con su porqué.

## Qué ofrece

- **Tests por tema** con corrección al momento y una explicación en cada pregunta.
- **Simulacro de examen** de 30 preguntas y 60 minutos con todos los temas.
- **Repasar** reúne las preguntas falladas y las marcadas hasta que se aciertan.
- **Mi progreso** con racha, meta diaria, logros y rangos.
- **Buscador** de asignaturas, temas y preguntas (Ctrl/Cmd + K).
- Tema claro y oscuro, atajos de teclado y sin cuentas ni registro.

| Asignatura | Resultados | Mi progreso |
|---|---|---|
| ![Página de una asignatura con sus temas y el simulacro](docs/capturas/asignatura.png) | ![Pantalla de resultados](docs/capturas/resultados.png) | ![Mi progreso](docs/capturas/progreso.png) |

En el móvil, con la barra de navegación abajo:

<p align="center"><img src="docs/capturas/movil.png" width="300" alt="Página de una asignatura en el móvil, con la barra de navegación inferior"></p>

## Retos técnicos

**Experiencia**

- **Audio sin archivos de audio.** Las preguntas de escucha de Inglés las lee la voz del navegador (Web Speech API), con barra de progreso estimada y afinada con la posición real de la voz, modo más despacio y transcripción.
- **La explicación siempre a la vista, sin saltos.** Ninguna interacción mueve la página, salvo al responder: si la explicación queda fuera de la pantalla, baja lo justo para mostrarla, sin que la tape la barra fija (WCAG 2.4.11).
- **Sin parpadeo de tema.** Un script mínimo en el `<head>` aplica el tema claro u oscuro antes de pintar.
- **Compartir nativo.** En el móvil se abre el menú del sistema (con un arreglo para Safari) y en el escritorio son enlaces normales, sin cargar scripts de redes sociales.

**Datos y lógica**

- **Progreso que no se pierde.** Sin cuentas ni servidor, con versión, validación al leer, respaldo automático si algo se daña y soporte para varias pestañas abiertas.
- **Buscador de 2.300 preguntas sin librerías.** Un índice generado en la build que se descarga al escribir la tercera letra, se valida y muestra un esqueleto mientras llega. Ignora tildes y mayúsculas y resalta el texto original.
- **Constelaciones generadas.** Cada asignatura dibuja la suya con un árbol de expansión mínima y una semilla sacada de su nombre, y el mapa se pagina según el ancho de pantalla.
- **Una nota que no engaña.** Se trunca y nunca redondea a favor, y el simulacro reparte las preguntas por turnos para que todos los temas pesen igual.

<details>
<summary>Código: leer lo guardado sin fiarse</summary>

```ts
const version = esObjeto(sobre) ? sobre['v'] : undefined;
if (esEntero(version, 1) && version > esquema.version) {
  // Viene de una versión más nueva de la web: no se toca y se trabaja en memoria.
  bloqueadas.add(clave);
  avisar('version-nueva', esquema.nombre);
  return esquema.vacio();
}

const saneado = esObjeto(sobre) && version === esquema.version ? esquema.sanear(sobre['d']) : null;
if (saneado !== null && saneado.descartados === 0) return saneado.valor;

// Dañado del todo o en parte: se respalda el original y se sigue con lo que se haya salvado.
const valor = saneado?.valor ?? esquema.vacio();
if (respaldar(clave, texto, esquema.nombre)) escribir(esquema, valor);
else memoria.set(clave, valor);
return valor;
```
</details>

<details>
<summary>Código: unir las estrellas de una constelación (algoritmo de Prim)</summary>

```ts
const unidos = [0];
const aristas: [number, number][] = [];
while (unidos.length < n) {
  let mejor: { a: number; b: number; d: number } | null = null;
  for (const a of unidos) {
    for (let b = 0; b < n; b++) {
      if (unidos.includes(b)) continue;
      const d = Math.hypot(pts[a][0] - pts[b][0], pts[a][1] - pts[b][1]);
      if (mejor === null || d < mejor.d) mejor = { a, b, d };
    }
  }
  if (mejor === null) break;
  aristas.push([mejor.a, mejor.b]);
  unidos.push(mejor.b);
}
```
</details>

**Calidad**

- **Preguntas sin trampa.** Un validador en cada build avisa si la correcta tiende a ser la más larga (comparando con lo esperable por azar) y rechaza imágenes SVG con scripts o referencias externas.
- **394 pruebas** de la lógica y del validador, que se ejecutan en milisegundos sin navegador.

---

**Rendimiento y SEO**

El SEO y el rendimiento se trabajaron con mucho cuidado, página a página, para exprimir cada detalle. Funcionó: al buscar «tests interactivos daw dam», Google muestra la web la primera, tanto en la vista creada con IA como en los resultados (octubre de 2026).

![Búsqueda «tests interactivos daw dam» en Google, con benrios.dev en primer lugar en la vista creada con IA y en los resultados](docs/capturas/google.png)

Lighthouse en la portada (PageSpeed Insights):

| Móvil | Escritorio |
|---|---|
| ![Móvil: 100 de rendimiento, 96 de accesibilidad, 100 de buenas prácticas y 100 de SEO](docs/capturas/lighthouse-movil.png) | ![Escritorio: 100 en rendimiento, accesibilidad, buenas prácticas y SEO](docs/capturas/lighthouse-escritorio.png) |

- **Una sola regla para indexar.** Una lista decide qué páginas van al sitemap, y todas las demás reciben `noindex, follow` solas, así que el sitemap y las etiquetas nunca se contradicen. Se indexan la portada, las asignaturas y los temas. Repasar, Mi progreso y el simulacro no, porque dependen de cada usuario.
- **Sitemap propio, sin dependencias**, con la fecha de modificación sacada del último commit de cada página.
- **Metadatos completos.** Canonical en cada página, Open Graph con una imagen propia de 1200×630, miga de pan con datos estructurados, un solo `h1` por página y descripciones de tema generadas de menos de 160 caracteres.
- **Sin páginas pobres.** No se crea una página por pregunta, para no llenar el índice de contenido escaso. Los enunciados de cada tema están en su página.

## Errores que encontramos

Algunos fallos reales del desarrollo, con su causa y cómo se resolvieron.

- **El validador daba por repetidas opciones distintas.** `new HashMap<>()` y `new HashMap()` salían como duplicadas, porque al normalizar se quitaban todos los signos. Ahora el texto se compara normalizado, pero el código en línea se compara tal cual. Una prueba cubre el caso.
- **27 falsos avisos de sesgo en un solo tema.** En las preguntas de verdadero o falso, «Verdadero» siempre es más larga que «Falso», así que el control de longitud saltaba sin motivo. Se sacaron de ese cálculo y tienen su propio control: avisar si la misma respuesta es la correcta en más del 70 %.
- **Un fallo acertado seguía en Repasar.** El progreso solo se registraba al terminar el test, y quien salía a medias no veía el cambio. Se separó en dos operaciones: anotar cada respuesta al momento y sumar las cifras al terminar, para no contar nada dos veces.
- **Un oyente de `scroll` por cada pregunta.** El botón de compartir de cada pregunta registraba un oyente al pintarse y nunca lo quitaba, así que se acumulaban durante el test. Se corrigió quitándolo cuando el panel salía del DOM y, al revisar coste y beneficio, se retiró la función entera.
- **«Copiar» en Safari perdía el texto.** Con `navigator.share({ text, url })`, la opción de copiar de iOS repetía la dirección y descartaba el texto. Ahora el texto y la dirección van juntos en `text`, y el menú del sistema solo se usa en pantallas táctiles (`pointer: coarse`).
- **Los enlaces de la barra se montaban sobre la marca en el móvil.** Una regla más específica (`.top nav a.desk`) ganaba a la que los escondía. La regla de ocultar pasó a usar el mismo selector.
- **Una pestaña repintaba la otra con cada respuesta.** Los cambios hechos en otra pestaña se aplicaban y se fundían en cuanto llegaban. Ahora se aplican una sola vez, cuando el usuario vuelve a ver esa página.
- **La analítica no contaba visitas.** El script puesto a mano enviaba los datos a un dominio que respondía 404 sin CORS. Se cambió al punto de recogida del propio dominio y se comprobó en la pestaña Red que la petición devuelve 204.
- **Accesibilidad del 94 al 96 en móvil.** PageSpeed avisó de que ninguna página tenía `<main>` y de que la etiqueta de versión tenía un contraste de 4,2:1. Se añadió `<main>` en el layout común, ajustando el selector que daba el ancho para que los titulares siguieran a todo el ancho, y el color pasó a mezclarse con el del texto para llegar a 5,1:1.

## Stack y cifras

- **Astro** con salida estática, **TypeScript** y CSS propio, sin framework de UI.
- **Vitest** para las pruebas.
- 10 asignaturas y 2.321 preguntas.

Las decisiones de diseño más importantes, con su porqué, están en [docs/decisiones.md](docs/decisiones.md).

## Sin backend, de momento

La web es estática a propósito. El progreso vive en el navegador, así que no hace falta registrarse ni se guardan datos personales, carga rápido y se aloja en un hosting sencillo, sin un servidor que mantener ni proteger. Para empezar, era lo más simple que funcionaba.

No descarto una versión con backend en el futuro, por ejemplo con cuentas para sincronizar el progreso entre dispositivos. La lógica ya está separada de la interfaz y todo el guardado pasa por una sola pieza, así que cambiar dónde se guardan los datos no obligaría a rehacer la web.

## Código fuente

El código fuente es privado. Este repositorio es un escaparate del proyecto. Todos los derechos reservados.

Hecho por [BenriosDev](https://benrios.dev).
