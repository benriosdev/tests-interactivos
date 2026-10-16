# Tests DAW y DAM

Tests interactivos para repasar las asignaturas de los ciclos de Desarrollo de Aplicaciones Web y Multiplataforma, con preguntas revisadas y explicadas.

**[Probar la web →](https://benrios.dev/tests/)**

![Portada con el mapa de constelaciones](docs/capturas/portada.gif)

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

## Retos técnicos

- **Progreso que no se pierde.** Sin cuentas ni servidor: todo vive en el navegador, con versión, validación al leer y respaldo automático si algo se daña.
- **Constelaciones generadas.** Cada asignatura dibuja la suya con un árbol de expansión mínima y una semilla sacada de su nombre, así que siempre sale igual.
- **Buscador sin tildes.** Encuentra «funcion» en «Función» y resalta el texto original exacto.
- **Preguntas sin trampa.** Un validador en cada build avisa si la respuesta correcta tiende a ser la más larga.
- **Sin saltos.** Ninguna interacción mueve la página, y el foco vuelve siempre al mismo control.

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

## Stack y cifras

- **Astro** con salida estática, **TypeScript** y CSS propio, sin framework de UI.
- **Vitest**, con 394 pruebas de la lógica y del validador.
- 10 asignaturas y 2.321 preguntas.

Las decisiones de diseño más importantes, con su porqué, están en [docs/decisiones.md](docs/decisiones.md).

## Código fuente

El código fuente es privado. Este repositorio es un escaparate del proyecto. Todos los derechos reservados.

Hecho por [BenriosDev](https://benrios.dev).
