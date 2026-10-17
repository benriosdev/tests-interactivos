# Decisiones de diseño

Una selección de las decisiones técnicas que dieron forma a la web, en formato ADR corto: contexto, decisión, alternativas descartadas y consecuencias.

### 1. Web estática con Astro y TypeScript, sin framework de UI

**Contexto.** Es una web de contenido (asignaturas, temas y preguntas) con interacción concentrada en el test, alojada en un hosting compartido sin servidor de aplicaciones.

**Decisión.**
- Astro con salida estática y base `/tests/`: cada asignatura y cada tema es una página HTML generada en la build.
- La interactividad va en módulos TypeScript pequeños, sin variables globales, que solo se cargan en las páginas que los usan.
- Las piezas de interfaz se crean con un helper mínimo (`h()`) sobre la API del DOM, y el texto que viene de datos se pinta como nodos de texto, nunca como HTML.

**Descartado.** React o Svelte con islas: para el volumen de interacción, añadían peso y un ciclo de vida que mantener.

**Consecuencias.** Páginas rápidas e indexables, con muy poco JavaScript. A cambio, el estado de cada pantalla se gestiona a mano.

### 2. Progreso sin cuentas, en `localStorage` versionado

**Contexto.** Sin servidor ni registro, todo el progreso vive en el navegador, que puede fallar, llenarse o editarse a mano.

**Decisión.**
- Tres claves con prefijo propio (progreso, ajustes y test en curso), cada una envuelta en `{ v, d }` con su versión de esquema.
- Al leer, cada esquema sanea los datos campo a campo y cuenta lo descartado. Si algo se pierde, el original se respalda en otra clave antes de sustituirlo.
- Si los datos vienen de una versión más nueva, la clave se bloquea y se trabaja en memoria, sin sobrescribir nada.
- `QuotaExceededError`, el modo privado y el bloqueo de almacenamiento se convierten en avisos para la interfaz.
- Cada escritura parte de lo último guardado (`actualizar`), por si hay otra pestaña abierta.

**Descartado.** Cuentas con backend (fricción y datos personales que proteger) e IndexedDB (más complejo sin necesitar su capacidad).

**Consecuencias.** El progreso no pasa de un dispositivo a otro. A cambio, nada se borra en silencio, y el almacenamiento se inyecta, así que se prueba con uno falso.

### 3. Lógica pura sin DOM, con las reglas en un solo módulo

**Contexto.** El test tiene muchos casos (corrección, tiempo, simulacro, rachas, logros) y es fácil romperlos al tocar la interfaz.

**Decisión.**
- Crear una sesión, responder, entregar, calcular la nota y registrar el progreso son funciones puras que reciben el estado y devuelven uno nuevo.
- Las reglas (nota mínima, tamaño y duración del simulacro, metas diarias) viven en un único módulo, `reglas.ts`.
- Barajado Fisher-Yates con un generador mulberry32 con semilla, reproducible en las pruebas.
- La nota se trunca a un decimal (un 69,5 % es un 6,9, no un 7,0) y «superado» se calcula con enteros (`aciertos * 10 >= 7 * total`), así que siempre coincide con la nota que se ve.
- Cada intento tiene un id y nunca se registra dos veces.

**Descartado.** Lógica repartida en los manejadores de eventos.

**Consecuencias.** Unas 400 pruebas con Vitest que se ejecutan en menos de un segundo sin navegador. La interfaz solo pinta y llama.

### 4. Ninguna interacción mueve el scroll

**Contexto.** En muchas webs de tests la página salta al responder, filtrar o cambiar de pestaña, y se pierde el sitio.

**Decisión.**
- Cada repintado devuelve el foco al mismo control y no desplaza la página. Los cambios de contenido se funden y las esperas muestran un esqueleto.
- Todo respeta `prefers-reduced-motion`.
- **Excepción:** al responder, si la explicación queda fuera de la pantalla, la página baja lo justo para mostrarla, con `scroll-padding-top` para que no la tape la barra fija (WCAG 2.4.11). El foco se queda en la opción elegida.

**Descartado.** Un panel inferior fijo con la corrección, porque tapaba las opciones y apretaba las explicaciones con código.

**Consecuencias.** La página es predecible y se usa bien con teclado y lector de pantalla. La excepción se decidió tras revisar estudios de usabilidad sobre la atención en la primera pantalla.

### 5. Estudiar no es examinarse

**Contexto.** La misma web sirve para aprender un tema con calma y para ensayar un examen.

**Decisión.**
- **Tests por tema:** corrección inmediata con explicación, y la respuesta queda fija. El modo examen (reloj) está apagado por defecto y nunca se recuerda encendido.
- **Simulacro:** 30 preguntas repartidas por turnos entre todos los temas (ninguno tiene más de una pregunta de diferencia) y 60 minutos. Se puede cambiar de respuesta y moverse libremente, y las explicaciones llegan al entregar.
- **Tiempo:** con reloj cuenta el tiempo real; sin reloj, las pausas de más de 5 minutos no suman.
- Se supera con un 7 y lo no respondido cuenta como fallo, sin restar.

**Consecuencias.** Dos modos con reglas distintas pero el mismo motor, que solo cambia por la configuración de la sesión.

### 6. Un validador de contenido en cada build

**Contexto.** Más de 2.300 preguntas en Markdown no se pueden revisar a mano en cada cambio.

**Decisión.**
- La build falla si hay errores: más de una correcta o ninguna, entre dos y cuatro opciones, explicación obligatoria, ids con formato fijo y sin duplicados, temas declarados sin archivo o campos desconocidos.
- **Sesgo de longitud:** por tema, avisa si la correcta es la opción más larga más de lo esperable por azar más 15 puntos (40 % con cuatro opciones, 65 % con dos). Las de verdadero o falso tienen su propio control: aviso si la misma respuesta es la correcta en más del 70 %.
- **Imágenes:** solo rutas locales, con un tamaño máximo, y los SVG se rechazan si llevan scripts, manejadores de eventos, `foreignObject` o referencias externas.

**Descartado.** Un umbral fijo para el sesgo, que daba falsos avisos donde el azar ya es del 50 %.

**Consecuencias.** Los errores se detectan antes de publicar, y los avisos (que no bloquean) se revisan a mano.

### 7. Buscador sin librerías, con índice bajo demanda

**Contexto.** Buscar entre asignaturas, temas y miles de preguntas sin servidor de búsqueda.

**Decisión.**
- La build genera un índice JSON (id, asignatura, tema y enunciado en texto plano, sin opciones ni explicaciones).
- Se descarga la primera vez que la consulta tiene tres letras, se valida su forma antes de usarlo y mientras llega se ve un esqueleto. Si falla, se avisa y se reintenta al reabrir.
- La búsqueda pliega tildes y mayúsculas (`normalize('NFD')`), exige todas las palabras y trata los números como enteros («tema 1» no encuentra el 10).
- El resaltado se calcula sobre el texto plegado pero marca el original, con nodos de texto y `<mark>`, sin HTML.

**Descartado.** Pagefind o MiniSearch, que solo compensan con mucho más volumen, y una página por pregunta, que llenaría el índice de Google de contenido escaso.

**Consecuencias.** Cero dependencias y nada que cargar hasta que alguien busca.

### 8. SEO con una sola fuente de verdad

**Contexto.** Hay páginas que deben indexarse (portada, asignaturas y temas) y otras que dependen de cada usuario (Repasar, Mi progreso, simulacro, repasos).

**Decisión.**
- Una sola función, `paginasIndexables()`, genera el sitemap. El layout pone `noindex, follow` a toda página que no esté en esa lista, así que el sitemap y las etiquetas nunca se contradicen.
- Sitemap propio sin dependencias, con `lastmod` sacado del último commit de cada fuente.
- Canonical en cada página, Open Graph con una imagen propia de 1200×630, miga de pan con datos estructurados `BreadcrumbList`, un solo `h1` y descripciones de tema generadas de menos de 160 caracteres.

**Descartado.** `@astrojs/sitemap` y mantener el sitemap a mano.

**Consecuencias.** Una página nueva no se indexa hasta añadirla a la lista, que es el lado seguro.

### 9. Identidad propia y cielo quieto

**Contexto.** La web necesitaba personalidad sin distraer de estudiar, en tema claro y oscuro.

**Decisión.**
- Colores como tokens con `light-dark()`, y un tema elegido que se aplica con un script mínimo en el `<head>` antes de pintar, sin parpadeo.
- Un cielo de fondo con capas fijas que nunca se mueven: estrellas en oscuro y nubes en claro.
- En la portada, cada asignatura es una constelación. Las que no tienen figura dibujada la generan: puntos separados al menos una distancia mínima y unidos con un árbol de expansión mínima (Prim), con una semilla sacada del nombre (hash FNV-1a), así que siempre sale la misma.
- Contraste mínimo de 4,5:1 en el texto pequeño, mezclando el acento con el color del texto (`color-mix()`) donde no llegaba.

**Descartado.** Galaxias, brillos según la nota y estrellas animadas.

**Consecuencias.** La metáfora del cielo da identidad sin pedir atención, y el mapa se adapta solo a cada asignatura nueva.

### 10. Audio de escucha con la voz del navegador

**Contexto.** Las preguntas de escucha de Inglés necesitaban audio, sin archivos que grabar, alojar y mantener.

**Decisión.**
- El enunciado lleva la transcripción en un bloque `audio` (británico) o `audio-us` (americano), y la web la lee con la Web Speech API (`SpeechSynthesisUtterance`).
- La barra de progreso se estima por la longitud del texto y se afina con los eventos `boundary` de la voz.
- Botón «Más despacio», transcripción oculta hasta corregir y atajo de teclado. Enunciado y opciones llevan `lang="en"` para lectores de pantalla y traductores.

**Descartado.** Archivos de audio y servicios externos de texto a voz.

**Consecuencias.** Cero archivos y cero dependencias. La voz depende del sistema y la barra no se puede arrastrar, porque la API no permite saltar.

### 11. Repasar se pone al día con cada respuesta

**Contexto.** Una pregunta acertada en un repaso seguía en Repasar si se salía sin terminar, porque el progreso solo se registraba al final.

**Decisión.**
- Dos operaciones separadas: `anotarRespuesta` actualiza al momento si una pregunta queda fallada, y `registrarResultado` suma las cifras (aciertos, días, racha, logros) solo al terminar.
- En el simulacro todo cuenta al entregar, porque no enseña si aciertas.

**Descartado.** Dejarlo como estaba y explicarlo con un aviso.

**Consecuencias.** Repasar refleja siempre la realidad y ninguna cifra se cuenta dos veces. Una prueba recorre el caso completo.

### 12. Analítica sin cookies

**Contexto.** Hacía falta saber cuánta gente entra, sin un banner de cookies nada más llegar.

**Decisión.** Cloudflare Web Analytics, que no usa cookies ni almacenamiento en el navegador, sin eventos y cargado solo en la build de producción.

**Descartado.** Google Analytics 4 (obliga al banner, envía datos fuera de la UE y pesa más) y Plausible (de pago o alojada).

**Consecuencias.** Sin banner y sin datos personales. A cambio, no hay métricas de uso detalladas, como cuántos terminan un test.

### 13. Quitar lo que complica más de lo que aporta

**Contexto.** En una misma tanda se añadieron un botón para compartir cada pregunta y diálogos que preguntaban antes de sustituir un test a medias al abrir un enlace.

**Decisión.** Se revisaron coste y beneficio ese mismo día y se retiraron. Los diálogos necesitaban redirecciones y detectar el origen de la visita (`document.referrer`) para evitar algo de poca gravedad, y trataban distinto un enlace que empezar un test desde la propia web.

**Consecuencias.** Menos código que mantener y probar. Si algo aporta menos de lo que cuesta, se quita aunque ya esté hecho, y no se vuelve a proponer sin datos que lo justifiquen.
