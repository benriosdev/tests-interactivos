# Decisiones de diseño

Una selección de las decisiones que dieron forma a la web y su porqué.

### 1. Web estática con Astro y TypeScript, sin framework de UI
Astro genera páginas estáticas y la interactividad va en módulos TypeScript pequeños, sin variables globales ni React. Cada página llega ya construida, carga rápido y el navegador recibe solo el JavaScript que hace falta.

### 2. Progreso sin cuentas
Todo se guarda en `localStorage`, en claves con prefijo y versión. Lo leído se trata como entrada no fiable: se comprueba entero, lo dañado se respalda antes de sustituirlo y, si el navegador no deja guardar, se sigue en memoria con un aviso. Sin cuentas no hay datos personales que proteger ni fricción para empezar.

### 3. Lógica sin DOM, con pruebas
Crear un test, responder, entregar, calcular la nota y registrar el progreso son funciones puras probadas con Vitest, y las reglas viven en un único archivo. Así se cubren detalles que a mano se escapan, como que un 69,5 % sea un 6,9 y no un 7,0, o que el mismo intento nunca se registre dos veces.

### 4. Ninguna interacción mueve el scroll
Cada repintado devuelve el foco al mismo control sin desplazar la página, y donde hay espera se ve un esqueleto. La única excepción es al responder: si la explicación queda fuera de la pantalla, la página baja lo justo para mostrarla (WCAG 2.2, 2.4.11). *Descartado:* un panel fijo de corrección, que tapaba las opciones.

### 5. Estudiar no es examinarse
Los tests por tema corrigen al momento y el reloj está apagado por defecto. El simulacro sí es un examen: 30 preguntas repartidas entre todos los temas, 60 minutos y explicaciones al final. Se supera con un 7 y lo no respondido cuenta como fallo, sin restar.

### 6. Contra el sesgo de longitud
El validador de cada build avisa si, en un tema, la correcta es la opción más larga más de lo esperable por azar más 15 puntos (40 % con cuatro opciones, 65 % con dos). Las de verdadero o falso tienen su propio control, porque «Verdadero» siempre es más larga.

### 7. Identidad propia y cielo quieto
Tema claro y oscuro, con un cielo de fondo que nunca se mueve: estrellas en oscuro y nubes en claro. En la portada cada asignatura es una constelación y cada tema una estrella que cambia con su estado. *Descartado:* galaxias, brillos según la nota y estrellas animadas, que compiten con el contenido.

### 8. Repasar se pone al día con cada respuesta
Una pregunta acertada sale de Repasar al momento, aunque no se termine el test. Las cifras (aciertos, racha, logros) se suman solo al terminar, para no contar nada dos veces.

### 9. Analítica sin cookies
Cloudflare Web Analytics, sin cookies ni almacenamiento, así que no hace falta banner y quien entra va directo al test. *Descartado:* Google Analytics 4, que obliga al banner y pesa más.

### 10. Quitar lo que complica más de lo que aporta
Se probaron un botón para compartir cada pregunta y un diálogo antes de sustituir un test a medias, y se retiraron el mismo día: añadían lógica y mantenimiento para evitar problemas menores. Si algo aporta menos de lo que cuesta, se quita aunque ya esté hecho.
