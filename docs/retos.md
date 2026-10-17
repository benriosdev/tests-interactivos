# Retos técnicos

Los problemas más interesantes que hubo que resolver y los errores que salieron por el camino. Las decisiones de diseño, con su porqué, están en [decisiones.md](decisiones.md).

## Experiencia

- **Audio sin archivos de audio.** Las preguntas de escucha de Inglés las lee la voz del navegador (Web Speech API), con barra de progreso estimada y afinada con la posición real de la voz, modo más despacio y transcripción.
- **La explicación siempre a la vista, sin saltos.** Ninguna interacción mueve la página, salvo al responder: si la explicación queda fuera de la pantalla, baja lo justo para mostrarla, sin que la tape la barra fija (WCAG 2.4.11).
- **Sin parpadeo de tema.** Un script mínimo en el `<head>` aplica el tema claro u oscuro antes de pintar.
- **Compartir nativo.** En el móvil se abre el menú del sistema (con un arreglo para Safari) y en el escritorio son enlaces normales, sin cargar scripts de redes sociales.

## Datos y lógica

- **Progreso que no se pierde.** Sin cuentas ni servidor, con versión, validación al leer, respaldo automático si algo se daña y soporte para varias pestañas abiertas.
- **Buscador de 2.300 preguntas sin librerías.** Un índice generado en la build que se descarga al escribir la tercera letra, se valida y muestra un esqueleto mientras llega. Ignora tildes y mayúsculas y resalta el texto original.
- **Constelaciones generadas.** Cada asignatura dibuja la suya con un árbol de expansión mínima y una semilla sacada de su nombre, y el mapa se pagina según el ancho de pantalla.
- **Una nota que no engaña.** Se trunca y nunca redondea a favor, y el simulacro reparte las preguntas por turnos para que todos los temas pesen igual.

#### Código: leer lo guardado sin fiarse

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

#### Código: unir las estrellas de una constelación (algoritmo de Prim)

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

## Calidad

- **Preguntas sin trampa.** Un validador en cada build avisa si la correcta tiende a ser la más larga (comparando con lo esperable por azar) y rechaza imágenes SVG con scripts o referencias externas.
- **396 pruebas** de la lógica y del validador, que se ejecutan en milisegundos sin navegador.

## Rendimiento, accesibilidad y SEO

**100 en las cuatro categorías de Lighthouse (rendimiento, accesibilidad, buenas prácticas y SEO), en móvil y en escritorio.** Medido con PageSpeed Insights en la portada, en octubre de 2026.

| Móvil | Escritorio |
|---|---|
| ![Móvil: 100 en rendimiento, accesibilidad, buenas prácticas y SEO](capturas/lighthouse-movil.png) | ![Escritorio: 100 en rendimiento, accesibilidad, buenas prácticas y SEO](capturas/lighthouse-escritorio.png) |

No salió solo: la accesibilidad en móvil empezó en 94 y llegó a 100 corrigiendo cada aviso (están más abajo, en «Errores que encontramos»).

### Rendimiento

- **HTML estático y poco JavaScript.** Cada página llega construida desde la build, sin framework de UI, y cada script solo se carga en las páginas que lo usan.
- **Recursos propios.** Fuentes, iconos e imágenes se sirven desde la propia web; lo único de fuera es el script de la analítica, que solo va en producción. Y la build falla si algún HTML lleva scripts, estilos o manejadores de eventos en línea.
- **Datos bajo demanda.** Las preguntas de cada asignatura van en su propio JSON, que se descarga al abrir un test, y el índice del buscador solo cuando alguien escribe tres letras.
- **Una sola fuente, precargada.** Onest variable en `woff2`, servida desde la propia web y precargada, para que el texto no cambie al llegar.
- **Archivos con hash en el nombre.** El CSS y el JS cambian de nombre en cada build, así que el navegador puede guardarlos sin miedo a servir una versión vieja.

### Accesibilidad

- **Estructura clara.** Un punto de referencia `<main>` en cada página, navegación con nombre, un solo `h1` y migas de pan.
- **Contraste suficiente.** El texto pequeño llega al menos a 4,5:1 en los dos temas (WCAG 1.4.3), mezclando el color de acento con el del texto donde no llegaba.
- **Áreas táctiles de 24 px o más y sin solapes** (WCAG 2.5.8), también en el mapa de constelaciones del móvil.
- **Teclado completo.** Foco siempre visible, atajos para responder (1 a 4 e Intro), marcar (M), escuchar (L) y ver los atajos (?), y nada fijo que tape lo que tiene el foco (WCAG 2.4.11).
- **Lectores de pantalla.** Las estrellas del mapa dicen tema, estado y acción; los avisos se anuncian solos; el inglés lleva `lang="en"`, y las imágenes de las preguntas tienen texto alternativo que no da la respuesta.
- **Movimiento reducido.** Todas las transiciones respetan `prefers-reduced-motion`.

### SEO

- **Una sola regla para indexar.** Una lista decide qué páginas van al sitemap, y todas las demás reciben `noindex, follow` solas, así que el sitemap y las etiquetas nunca se contradicen. Se indexan la portada, las asignaturas y los temas. Repasar, Mi progreso y el simulacro no, porque dependen de cada usuario.
- **Sitemap propio, sin dependencias**, con la fecha de modificación sacada del último commit de cada página.
- **Metadatos completos.** Canonical en cada página, Open Graph con una imagen propia de 1200×630, miga de pan con datos estructurados `BreadcrumbList`, un solo `h1` por página y descripciones de tema generadas de menos de 160 caracteres.
- **Sin páginas pobres.** No se crea una página por pregunta, para no llenar el índice de contenido escaso. Los enunciados de cada tema están en su página.
- **Resultado.** Al buscar solo «tests daw», Google pone la web la primera en la vista creada con IA, en el panel de fuentes y en los resultados (octubre de 2026).

![Búsqueda «tests daw» en Google, con benrios.dev en primer lugar en la vista creada con IA, en el panel de fuentes y en los resultados](capturas/google-tests-daw.png)

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
- **Accesibilidad del 96 al 100 en móvil.** Quedaba el aviso de áreas táctiles, con dos casos. En el mapa encogido del móvil, el enlace con el nombre de la asignatura tapaba por abajo la zona táctil de las estrellas más altas, así que tocar una estrella podía abrir la asignatura. Ahora el hueco del nombre se calcula midiendo en la página el alto real de los nombres (con su fuente y el ancho de su columna), la fila crece sin encoger la constelación y, por si acaso, las estrellas quedan por encima de los nombres. Además, los puntos de página del mapa caían detrás de la barra inferior fija del móvil: con esa barra a la vista, el resumen y los puntos pasan a una cabecera encima del mapa, y el HTML se genera en el orden en que se ve, para que el tabulador y el lector de pantalla sigan ese mismo orden. Para que el alto medido no cambie al llegar la fuente, ahora se precarga.
