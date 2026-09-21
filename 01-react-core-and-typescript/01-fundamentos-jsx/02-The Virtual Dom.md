# El DOM virtual: cómo React actualiza la pantalla

## En una frase

El **DOM virtual** es un patrón: React describe la UI como un árbol de objetos en memoria, compara ese árbol con el del render anterior y aplica al DOM real solo las diferencias.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es JSX y en qué se transforma: [Introducción a JSX](01-Intro%20to%20JSX.md).
* Qué es el DOM: la representación en forma de árbol que el navegador construye de la página y que JavaScript puede modificar.

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Elemento de React:** objeto JavaScript simple que describe qué debe mostrarse (tipo, props y hijos). Cada expresión JSX produce uno.
* **Render:** que React ejecute los componentes para calcular qué UI debe mostrar. No toca el DOM.
* **Reconciliación:** proceso por el cual React compara el árbol de elementos nuevo con el anterior para decidir qué cambiar.
* **Commit:** la fase en la que React aplica esos cambios al DOM real.

-----

## El problema

Modificar el DOM a mano es costoso y propenso a errores:

1. **Cada cambio tiene costo.** Alterar nodos puede obligar al navegador a recalcular estilos y layout, y a volver a pintar.
2. **Hay que rastrear el estado a mano.** Con código imperativo, tú decides qué nodo crear, cambiar o borrar según cada combinación de datos. Al crecer la app, es fácil dejar la pantalla desincronizada con los datos.
3. **Es tentador rehacer de más.** La salida más simple, reconstruir toda la lista cuando cambia un elemento, descarta trabajo y también el estado del navegador (foco, texto escrito, posición del scroll).

Se busca poder escribir "esta es la UI para estos datos" y que alguien más resuelva cómo llegar a ella con pocos cambios. Ese es el trabajo de React, y el DOM virtual es el mecanismo conceptual que lo hace posible.

-----

## Cómo funciona

React separa **calcular** la UI de **modificar** el DOM:

```
 cambia el estado o las props
            |
            v
 1. RENDER: React ejecuta el componente
            y obtiene un árbol de elementos (nuevo)
            |
            v
 2. RECONCILIACIÓN: compara el árbol nuevo
    con el del render anterior
            |
            v
 3. COMMIT: aplica al DOM real solo
    las diferencias encontradas
            |
            v
 el navegador actualiza la pantalla
```

En la documentación de React estos pasos se llaman *desencadenar* (trigger), *renderizar* (render) y *confirmar* (commit).

### Qué es el "DOM virtual"

No es una copia del DOM ni una API que uses. Es una idea: mantener en memoria una representación de la UI y sincronizarla con el DOM real. En React esa representación son los **elementos de React** (lo que devuelve tu JSX) y, internamente, objetos llamados *fibers* que guardan información del árbol de componentes.

Un elemento de React es solo un objeto; no puede cambiar lo que ves. Por eso crearlo es barato: se puede recalcular sin tocar la pantalla. Analogía: editar un plano cuesta menos que mover paredes en una casa construida.

### Reconciliación

Al comparar el árbol nuevo con el anterior, React sigue reglas generales:

* Si un elemento cambia de **tipo** (por ejemplo, de `<div>` a `<section>`, o de un componente a otro), React descarta ese subárbol y crea uno nuevo.
* Si el tipo es el mismo, conserva el nodo y actualiza solo las propiedades que cambiaron.
* En listas, usa la **key** de cada elemento para saber cuál es cuál entre renders.

### Commit

Con las diferencias ya calculadas, React modifica el DOM real. Según su documentación, React solo cambia los nodos del DOM cuando hay diferencia entre renders. Los nodos sin cambios (como un `<input>` con texto escrito) no se tocan.

### Dos precisiones

* **Renderizar no es lo mismo que modificar el DOM.** Un componente puede ejecutarse (render) y no producir ningún cambio en el DOM.
* **Un render vuelve a ejecutar los hijos.** Por defecto, cuando un componente se renderiza de nuevo, también se renderizan los componentes que devuelve. Sigue siendo barato en el DOM porque el commit filtra lo que no cambió.

-----

## Ejemplo completo

Una lista de tareas donde se marca la primera como completada:

```jsx
import { useState } from 'react';

const initialTasks = [
  { id: 1, text: 'Leer sobre JSX', done: false },
  { id: 2, text: 'Escribir un componente', done: false },
  { id: 3, text: 'Practicar estado', done: false },
];

function TaskList() {
  const [tasks, setTasks] = useState(initialTasks);

  function toggle(id) {
    setTasks(
      tasks.map((task) =>
        task.id === id ? { ...task, done: !task.done } : task
      )
    );
  }

  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <label>
            <input
              type="checkbox"
              checked={task.done}
              onChange={() => toggle(task.id)}
            />
            {task.done ? <s>{task.text}</s> : task.text}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

Al marcar la primera tarea:

```
1. onChange llama a setTasks con un array NUEVO
   (el primer objeto es una copia con done: true)
2. React vuelve a ejecutar TaskList
   -> genera un árbol con los tres <li>
3. Reconciliación: compara con el árbol anterior
   usando las keys 1, 2 y 3
   -> el <li> con key 1 cambió (checked y <s>)
   -> los <li> con key 2 y 3 son idénticos
4. Commit: solo se actualiza el <li> con key 1
```

Punto clave: `TaskList` se ejecutó completo y generó los tres elementos, pero el DOM real solo cambió donde hubo diferencia.

-----

## Errores comunes

### "El DOM virtual hace a React más rápido que el DOM real"

**Qué pasa:** se toma como una ley que React supera al DOM directo.

**Por qué es una simplificación:** el DOM virtual **agrega** trabajo (crear y comparar árboles). Un código manual y bien optimizado que toque exactamente los nodos necesarios puede ser igual o más rápido. Lo que aporta React es que casi siempre obtienes actualizaciones razonablemente eficientes **sin escribirlas a mano**, con un modelo declarativo.

**Cómo pensarlo:** el valor principal es productividad y previsibilidad; el rendimiento es "suficientemente bueno por defecto", no "el más rápido posible".

### Listas sin `key`, o con una `key` inestable

**Qué pasa:** React advierte por consola; con `key={Math.random()}` las keys nunca coinciden entre renders y se recrea todo el DOM de la lista; con el índice del array, al reordenar o insertar aparecen errores sutiles (por ejemplo, el texto de un input queda asociado al elemento equivocado).

**Por qué:** la key es la identidad que React usa para emparejar elementos entre renders.

**Cómo se arregla:** usa un identificador estable y único entre hermanos, normalmente el `id` de tus datos.

```jsx
// Mal: cambia en cada render
<li key={Math.random()}>{task.text}</li>

// Bien: identidad estable
<li key={task.id}>{task.text}</li>
```

### Mutar el estado

**Qué pasa:** haces `tasks[0].done = true` y llamas a `setTasks(tasks)`. La pantalla puede no actualizarse.

**Por qué:** React decide si hay algo nuevo comparando el valor del estado con el anterior. Si le pasas el mismo array (misma referencia), no detecta un cambio. Además, el render debe ser un cálculo puro y no debe alterar objetos que ya existían.

**Cómo se arregla:** crea siempre un valor nuevo, como en `toggle` del ejemplo con `map` y el operador spread.

### Creer que renderizar es actualizar el DOM

**Qué pasa:** se intenta "evitar re-renders" pensando que cada uno cuesta un cambio en el DOM.

**Por qué:** un render es un cálculo en memoria; el DOM solo cambia en el commit y solo donde hay diferencias. Optimizar renders es un problema aparte y suele ser innecesario hasta que se mida un problema real.

-----

## Cuándo sí y cuándo no

**El DOM virtual conviene cuando:**

* La UI depende de datos que cambian y quieres describirla de forma declarativa.
* No quieres rastrear a mano qué nodos crear, cambiar o eliminar.

**No lo trates como:**

* Una herramienta de optimización que debas manejar. No lo manipulas directamente: escribes componentes y React se encarga.
* Una garantía de que todo será rápido. Renders muy pesados o listas enormes siguen necesitando análisis y, cuando se mide un problema, técnicas específicas.

-----

## Resumen en 5 líneas

1. El DOM virtual es un patrón: la UI se describe como un árbol de objetos en memoria.
2. Ante un cambio, React renderiza (calcula un árbol nuevo) sin tocar el DOM.
3. La reconciliación compara el árbol nuevo con el anterior.
4. El commit aplica al DOM real solo las diferencias.
5. Las `key` identifican elementos en listas y el estado nunca se muta: se reemplaza.

-----

## Para profundizar

<details>
<summary>Fiber</summary>

Fiber es el motor de reconciliación introducido en React 16. Según la documentación histórica de React, su objetivo principal es permitir el renderizado incremental. Cada *fiber* es un objeto interno con información del componente y su lugar en el árbol. Es un detalle de implementación: no lo usas directamente.

</details>

<details>
<summary>Batching</summary>

React agrupa varias actualizaciones de estado producidas en un mismo evento y las procesa en un solo render, en lugar de renderizar tras cada llamada al setter. Por eso, dentro de un handler, el estado que lees es la "instantánea" del render actual y no cambia tras llamar al setter.

</details>

<details>
<summary>Keys y reconciliación</summary>

En una lista, la key le dice a React a qué elemento del array corresponde cada componente, algo que importa cuando los elementos se mueven, se insertan o se eliminan. Las keys deben ser únicas entre hermanos (no hace falta que lo sean entre listas distintas), no deben cambiar y no deben generarse durante el render.

Una key también puede usarse para forzar el reinicio de un componente: si cambia, React lo trata como uno distinto y descarta su estado.

</details>

-----

## En entrevista

### Respuesta corta (junior)

El DOM virtual es una representación de la UI en memoria. Cuando cambia el estado, React calcula una UI nueva, la compara con la anterior y modifica el DOM real solo donde hay diferencias. Así el desarrollador describe cómo debe verse la UI y React se ocupa de actualizarla.

### Respuesta ampliada (semi-senior)

El DOM virtual no es una tecnología concreta sino un patrón. En React, son los elementos que devuelven los componentes y, internamente, los fibers. Un cambio de estado desencadena un render: React ejecuta los componentes y obtiene un árbol nuevo. Luego la reconciliación lo compara con el anterior (cambio de tipo implica recrear el subárbol; mismo tipo implica actualizar props; las listas se emparejan por key). En el commit, React aplica solo las diferencias al DOM.

Importa aclarar que no es "más rápido que el DOM": agrega trabajo de comparación, y lo que ofrece es un modelo declarativo con actualizaciones eficientes por defecto. También que renderizar (cálculo, debe ser puro) es distinto de hacer commit (tocar el DOM).

### Preguntas frecuentes de seguimiento

**¿Qué es el DOM virtual?**
Un patrón en el que la UI se mantiene como una representación en memoria que se sincroniza con el DOM real. En React son los elementos de React y los fibers internos.

**¿Es más rápido que el DOM real?**
No necesariamente. Añade trabajo de creación y comparación de árboles. Su ventaja es que evita escribir las actualizaciones a mano y, por lo general, produce cambios mínimos en el DOM.

**¿Qué es la reconciliación?**
El proceso de comparar el árbol de elementos nuevo con el anterior para decidir qué cambios aplicar al DOM. Se basa en el tipo de cada elemento y, en listas, en las keys.

**¿Para qué sirven las keys?**
Para que React identifique qué elemento del array corresponde a cada componente entre renders. Con keys estables, entiende bien inserciones, borrados y reordenamientos.

**¿Qué dispara un re-render?**
Principalmente un cambio de estado en el componente (o el render de su padre, que por defecto vuelve a renderizar a sus hijos). Los cambios de props llegan como consecuencia del render del padre.

**¿DOM virtual vs Shadow DOM?**
Son cosas distintas. El Shadow DOM es una tecnología del navegador para aislar estilos y variables dentro de web components. El DOM virtual es un patrón implementado en JavaScript por librerías como React.

-----

## Siguiente lección

Continúa con [JSX avanzado](03-Advanced%20JSX.md). Después verás cómo escribir [JSX de varias líneas en un componente](04-Use%20Multiline%20JSX%20in%20a%20Component.md).
