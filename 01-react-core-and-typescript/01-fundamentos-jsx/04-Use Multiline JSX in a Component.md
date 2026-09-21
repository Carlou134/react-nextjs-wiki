# JSX multilínea y lógica dentro de un componente

## En una frase

Un componente de función puede devolver JSX en varias líneas (con paréntesis en el `return`), calcular valores **antes** del `return`, usarlos como atributos o contenido, y definir manejadores de eventos que se pasan al JSX.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es JSX y cómo se insertan expresiones con llaves: [Intro to JSX](01-Intro%20to%20JSX.md) y [Advanced JSX](03-Advanced%20JSX.md).
* Que un componente de función es una función que devuelve JSX.

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Expresión JSX multilínea:** JSX que ocupa varias líneas.
* **Manejador de eventos (event handler):** función que se ejecuta en respuesta a una interacción, como un clic.
* **Prop:** dato que se pasa a un elemento o componente mediante un atributo JSX.

-----

## El problema

Los componentes reales no devuelven una sola línea. Devuelven estructuras anidadas, usan datos en sus atributos, deciden qué mostrar según una condición y reaccionan a la interacción del usuario. Hay que saber dónde va cada cosa: qué se escribe antes del `return`, qué va dentro del JSX y cómo se conectan los eventos.

-----

## Cómo funciona

### JSX multilínea con paréntesis

Para devolver este HTML desde un componente:

```html
<blockquote>
  <p>La simplicidad es la máxima sofisticación.</p>
  <cite>
    <a target="_blank" href="https://es.wikipedia.org/wiki/Leonardo_da_Vinci">
      Leonardo da Vinci
    </a>
  </cite>
</blockquote>
```

se escribe así:

```jsx
function Quote() {
  return (
    <blockquote>
      <p>La simplicidad es la máxima sofisticación.</p>
      <cite>
        <a target="_blank" href="https://es.wikipedia.org/wiki/Leonardo_da_Vinci">
          Leonardo da Vinci
        </a>
      </cite>
    </blockquote>
  );
}
```

Los paréntesis agrupan la expresión para que el JSX pueda empezar en la línea siguiente al `return`. Son una regla de JavaScript, no de React (ver "Errores comunes"). Con JSX en una sola línea no hacen falta:

```jsx
return <h1>Hola</h1>;
```

Además, el JSX devuelto debe tener **un único elemento raíz**. Si necesitas devolver varios elementos hermanos sin añadir un nodo extra al DOM, envuélvelos en un Fragment (`<>...</>`).

### Variables como atributos

Puedes usar valores de JavaScript en los atributos con llaves:

```jsx
const redPanda = {
  src: 'https://upload.wikimedia.org/wikipedia/commons/b/b2/Endangered_Red_Panda.jpg',
  alt: 'Panda rojo',
  width: '200px'
};

function RedPanda() {
  return (
    <div>
      <h1>Panda rojo</h1>
      <img
        src={redPanda.src}
        alt={redPanda.alt}
        width={redPanda.width}
      />
    </div>
  );
}
```

Cada `{...}` inserta el resultado de una expresión de JavaScript. Es el mismo mecanismo que se usa para el contenido entre etiquetas.

### Lógica antes del `return`

Un componente de función es una función normal: puede tener código antes del `return`. Ahí van los cálculos y las decisiones que el JSX necesita.

```jsx
function Price({ amount, quantity }) {
  const total = amount * quantity;

  return <p>Total: {total}</p>;
}
```

En un componente, el cálculo debe ser **puro**: con las mismas props debe dar el mismo resultado. Por eso no conviene calcular aquí valores como `Math.random()` o `Date.now()`: cambian en cada render.

### Condicionales dentro del componente

Las sentencias `if` no pueden ir dentro de las llaves del JSX, porque ahí solo caben expresiones. Se escriben antes del `return` y se inserta el resultado:

```jsx
function Greeting({ isMorning }) {
  let task;

  if (isMorning) {
    task = 'aprender React';
  } else {
    task = 'descansar';
  }

  return <h1>Hoy voy a {task}.</h1>;
}
```

Dentro del JSX sí puedes usar expresiones condicionales, como el operador ternario o `&&`, que se cubren en [Advanced JSX](03-Advanced%20JSX.md).

### Manejadores de eventos

Un manejador se define dentro del componente y se pasa como valor de un atributo de evento. Por convención su nombre empieza por `handle` seguido del evento:

```jsx
function Alert() {
  function handleMouseEnter() {
    alert('Deja de pasar el mouse por aquí.');
  }

  return <div onMouseEnter={handleMouseEnter}>Pasa el mouse</div>;
}
```

Para acciones cortas puedes escribirlo inline, con una función flecha:

```jsx
<button onClick={() => alert('Clic')}>Aceptar</button>
```

Ambas formas son equivalentes. Lo importante es que se pasa **la función**, sin ejecutarla: `onClick={handleClick}`, no `onClick={handleClick()}`. Los paréntesis la ejecutan durante el render, no en el clic.

-----

## Ejemplo completo

```jsx
const user = {
  name: 'Ana',
  avatar: 'https://example.com/ana.png',
};

export default function Welcome({ isLoggedIn }) {
  // Lógica antes del return
  const message = isLoggedIn
    ? `Hola, ${user.name}`
    : 'Inicia sesión para continuar';

  function handleClick() {
    alert(message);
  }

  return (
    <section>
      <img src={user.avatar} alt={user.name} width="80" />
      <h2>{message}</h2>
      <button onClick={handleClick}>Ver mensaje</button>
    </section>
  );
}
```

Aquí conviven las cuatro piezas: cálculo antes del `return`, atributos con variables, contenido con expresiones y un manejador pasado por referencia.

-----

## Errores comunes

### 1. `return` seguido de salto de línea sin paréntesis

```jsx
function Bad() {
  return
    <h1>Hola</h1>;
}
```

**Qué pasa:** el componente no muestra el JSX.
**Por qué:** JavaScript inserta automáticamente un punto y coma después de `return` cuando lo que sigue está en otra línea. La función devuelve `undefined` y el JSX queda como código inalcanzable.
**Solución:** abre el paréntesis en la misma línea del `return`: `return (` ... `);`.

### 2. Declarar variables dentro del `return`

```jsx
function Bad() {
  return (
    const n = 5;
    <h1>{n}</h1>
  );
}
```

**Qué pasa:** error de sintaxis.
**Por qué:** dentro de los paréntesis va una única expresión, no sentencias como `const` o `if`.
**Solución:** mueve la declaración antes del `return`.

### 3. Llamar al manejador en vez de pasarlo

```jsx
<button onClick={handleClick()}>Aceptar</button>
```

**Qué pasa:** la función se ejecuta en cada render, no al hacer clic. Si ese manejador actualiza el estado, puede provocar un bucle de renders.
**Por qué:** `handleClick()` es una llamada; su resultado (normalmente `undefined`) es lo que recibe `onClick`.
**Solución:** `onClick={handleClick}` o `onClick={() => handleClick()}`.

### 4. Usar un nombre de evento que no existe

`onHover` no es un evento de React; el atributo se ignora sin avisar. Para pasar el mouse por encima se usa `onMouseEnter`. Consulta la lista de eventos soportados en la documentación oficial.

-----

## En TypeScript

TypeScript infiere el tipo de retorno de un componente a partir del `return`, así que no hace falta anotarlo. Donde sí ayuda es en las props y en los eventos:

```tsx
type WelcomeProps = { isLoggedIn: boolean };

export default function Welcome({ isLoggedIn }: WelcomeProps) {
  function handleClick(event: React.MouseEvent<HTMLButtonElement>) {
    console.log(event.currentTarget);
  }

  return <button onClick={handleClick}>Ver</button>;
}
```

* Si un manejador se define inline dentro del atributo, TypeScript infiere el tipo del evento.
* Si se define aparte, hay que tiparlo tú (como arriba).
* `handleClick` y `handleClick()` son ambos código válido para el compilador: TypeScript no detecta ese error de ejecución.

Más detalle en [Props](../02-componentes-y-props/03-Props.md).

-----

## Cuándo sí y cuándo no

* **Manejador inline** para acciones de una línea; **función con nombre** cuando tiene varias líneas o se reutiliza.
* **Lógica antes del `return`** cuando el cálculo es largo o se usa varias veces. Si el JSX se vuelve difícil de leer por tantas condiciones, extrae un componente más pequeño.
* **No pongas efectos secundarios** (peticiones, timers, modificar el DOM) en el cuerpo del componente: el cuerpo debe ser un cálculo puro sobre props y estado.

-----

## Resumen en 5 líneas

1. El JSX multilínea en un `return` se envuelve en paréntesis abiertos en la misma línea del `return`.
2. Un componente devuelve un único elemento raíz (o un Fragment).
3. La lógica va antes del `return`; dentro del JSX solo caben expresiones entre llaves.
4. Los manejadores se definen en el componente, se nombran `handleAlgo` y se pasan como `onClick={handleAlgo}`.
5. Pasar `fn` registra la función; escribir `fn()` la ejecuta en el render.

-----

## Para profundizar

<details>
<summary>Por qué falla un `return` sin paréntesis</summary>

JavaScript tiene una regla de inserción automática de punto y coma (ASI). Si tras `return` hay un salto de línea, se interpreta `return;`. Todo lo que sigue es código que nunca se ejecuta. Los paréntesis lo evitan porque el `(` en la misma línea obliga al parser a seguir leyendo la expresión.

</details>

<details>
<summary>Inline vs. función con nombre</summary>

Según la documentación de React, definir el manejador inline o con nombre es equivalente. Los manejadores inline son cómodos para funciones cortas. La convención es definir los manejadores dentro del componente para que tengan acceso a sus props y su estado.

</details>

-----

## En entrevista

### Respuesta corta (junior)

Un componente de función puede tener código antes del `return`, y el `return` puede devolver JSX en varias líneas envuelto en paréntesis. Dentro del JSX se insertan valores con llaves, y los manejadores de eventos se definen en el componente y se pasan por referencia, por ejemplo `onClick={handleClick}`.

### Respuesta ampliada (semi-senior)

* **Paréntesis:** no son sintaxis de JSX sino una protección frente a la inserción automática de punto y coma de JavaScript.
* **Un solo nodo raíz:** JSX se transforma en objetos de JavaScript y una función no puede devolver dos; se usa un elemento contenedor o un Fragment.
* **Lógica antes del `return`:** las sentencias (`if`, `const`) no caben en el JSX, que solo admite expresiones. El cuerpo debe ser puro respecto a props y estado.
* **Manejadores:** se pasan como referencia; ejecutarlos en el JSX los dispara durante el render.
* **Tipado:** el compilador no distingue `fn` de `fn()`, pero sí valida la firma del manejador contra el tipo esperado por el atributo.

### Preguntas frecuentes de seguimiento

**1. ¿Son obligatorios los paréntesis en un `return` de JSX?**
No si el JSX empieza en la misma línea que el `return`. Son necesarios cuando empieza en la línea siguiente.

**2. ¿Puedo usar `if` dentro de las llaves de JSX?**
No, ahí solo caben expresiones. Usa un `if` antes del `return`, un ternario o `&&`.

**3. ¿Qué diferencia hay entre `onClick={fn}` y `onClick={fn()}`?**
La primera pasa la función para que React la llame en el clic. La segunda la ejecuta durante el render y pasa su resultado.

**4. ¿Por qué se devuelve un solo elemento raíz?**
Porque JSX se convierte en un objeto de JavaScript y una función devuelve un valor. Un Fragment permite agrupar sin añadir nodos al DOM.

**5. ¿Handler inline o con nombre?**
Son equivalentes en comportamiento. Inline para una línea; con nombre para lógica más larga o legibilidad.

**6. ¿Qué convención de nombres se usa?**
`handle` más el evento en el componente (`handleClick`) y `on` más el evento en el atributo (`onClick`).

-----

## Siguiente lección

Ya sabes escribir un componente con JSX, lógica y eventos. Ahora se pasa a componentes propios y sus props: [Tu primer componente](../02-componentes-y-props/01-Your%20First%20React%20Component.md).
