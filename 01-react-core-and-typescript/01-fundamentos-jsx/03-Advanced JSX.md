# JSX avanzado: JavaScript dentro del marcado

## En una frase

Dentro de JSX, las llaves `{}` abren una "ventana" a JavaScript: ahí van **expresiones** (valores, operadores, llamadas a funciones) para mostrar datos, fijar atributos, decidir qué renderizar y generar listas.

-----

## Antes de empezar

Conviene que ya conozcas:

* Qué es JSX y qué produce: [Introducción a JSX](01-Intro%20to%20JSX.md).
* Cómo React representa la UI internamente: [El Virtual DOM](02-The%20Virtual%20Dom.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Expresión:** código que produce un valor (`2 + 3`, `name`, `formatDate(d)`, `a ? b : c`). Se puede usar donde se espera un valor.
* **Sentencia:** instrucción que ejecuta una acción pero no produce un valor (`if`, `for`, `let x = 1`). No se puede usar donde se espera un valor.
* **Manejador de evento (event handler):** función que React ejecuta cuando ocurre un evento, como un clic.
* **`key`:** atributo especial que identifica cada elemento de una lista entre renders.

-----

## El problema

JSX se parece a HTML, pero un componente necesita más que texto fijo: mostrar datos que cambian, reaccionar a clics, mostrar u ocultar partes de la interfaz y repetir elementos a partir de un array.

Sin una forma de mezclar JavaScript con el marcado, todo sería estático. Además, hay diferencias de sintaxis con HTML que producen errores si no se conocen.

-----

## Cómo funciona

### `className` en lugar de `class`

En JSX, los atributos HTML y SVG se escriben en camelCase. El caso más frecuente es `class`, que se escribe `className`:

```jsx
<h1 className="big">Título</h1>
```

`class` es una palabra reservada de JavaScript y JSX se convierte en JavaScript, por eso React eligió `className` (el nombre de la propiedad del DOM). En el HTML final se renderiza como `class`.

Otros ejemplos: `stroke-width` pasa a `strokeWidth` y `for` (en `<label>`) pasa a `htmlFor`. Los atributos `data-*` y `aria-*` conservan el guion.

### Etiquetas autocerradas

Algunos elementos no tienen contenido, como `<img>`, `<input>` o `<br>`. En HTML la barra final es opcional. En JSX es **obligatoria**: todas las etiquetas deben cerrarse.

```jsx
// Correcto
<br />
<img src="foto.jpg" alt="Perfil" />

// Error de sintaxis
<br>
```

### Llaves `{}`: expresiones, no sentencias

Todo lo que está entre etiquetas JSX se interpreta como texto, no como código:

```jsx
<h1>2 + 3</h1>      {/* muestra el texto "2 + 3" */}
<h1>{2 + 3}</h1>    {/* muestra 5 */}
```

Las llaves indican "aquí empieza JavaScript". Solo aceptan **expresiones**:

```jsx
<p>{price * quantity}</p>
<p>{formatDate(today)}</p>
<p>{user.name.toUpperCase()}</p>
```

Una sentencia como `if` o `for` no produce un valor y no cabe entre llaves. Las llaves solo se pueden usar en dos lugares: como contenido entre etiquetas (`<h1>{name}</h1>`) y como valor de un atributo, justo después del `=` (`src={avatar}`). No sirven para el nombre de una etiqueta ni de un atributo.

Las llaves marcan el inicio y el fin de la inyección, igual que las comillas delimitan un texto. No son parte del JavaScript resultante.

### Variables y atributos

El código dentro de las llaves comparte el ámbito del resto del archivo, así que puede leer variables declaradas fuera del JSX:

```jsx
const name = 'Gerardo';
const greeting = <p>Hola, {name}</p>;
```

También sirve para atributos. Con llaves se pasa el **valor** de la variable; con comillas, un texto literal:

```jsx
const sideLength = '200px';
const pics = { panda: '/images/panda.jpg' };

<img
  src={pics.panda}
  alt="Panda"
  width={sideLength}
  height={sideLength}
/>

<img src="{pics.panda}" />   {/* error: pasa el texto literal "{pics.panda}" */}
```

Para pasar un objeto se usan **dobles llaves**: las externas abren JavaScript y las internas son el objeto. Es el caso típico de `style`, que exige propiedades en camelCase:

```jsx
<ul style={{ backgroundColor: 'black', color: 'pink' }}>
```

### Eventos (`onClick`)

Los manejadores se asignan con atributos que empiezan con `on` y siguen en camelCase (`onClick`, `onChange`, `onMouseOver`). En HTML se escriben en minúsculas (`onclick`).

El valor debe ser una **función**, que React ejecuta cuando ocurre el evento:

```jsx
function Photo() {
  function handleClick() {
    alert('Clic en la imagen');
  }

  return <img src="/foto.jpg" alt="Foto" onClick={handleClick} />;
}
```

Se **pasa** la función; no se **llama**. `onClick={handleClick()}` ejecutaría la función durante el render, no al hacer clic. Para pasar argumentos, se envuelve en una función flecha:

```jsx
<button onClick={() => alert('Hola')}>Saludar</button>
<button onClick={() => remove(id)}>Eliminar</button>
```

Por convención, los manejadores se nombran `handleAlgo`, y las props de función de tus propios componentes empiezan con `on` (`onSave`). Los eventos "burbujean" hacia los elementos padre; `e.stopPropagation()` detiene ese recorrido y `e.preventDefault()` cancela el comportamiento por defecto del navegador (por ejemplo, el envío de un formulario).

### Condicionales

Como `if` es una sentencia, no se puede escribir entre llaves:

```jsx
// Error de sintaxis
<h1>
  {
    if (purchase.complete) {
      'Gracias por tu compra'
    }
  }
</h1>
```

Hay tres formas de expresar condiciones.

**`if` fuera del JSX.** Se calcula el resultado en una variable y se inserta con llaves:

```jsx
function ConcertInfo({ price }) {
  let ticketInfo;

  if (price === 0) {
    ticketInfo = <h2>Entrada gratuita</h2>;
  } else {
    ticketInfo = <h2>Entrada: ${price}</h2>;
  }

  return (
    <div>
      <h1>Próximo concierto</h1>
      {ticketInfo}
    </div>
  );
}
```

Es la opción más legible cuando hay varias ramas.

**Operador ternario** (`condición ? A : B`). Es una expresión, así que cabe entre llaves. Sirve cuando hay **dos** resultados posibles:

```jsx
<h1>{age >= 18 ? 'Adulto' : 'Menor'}</h1>
```

**Operador `&&`.** Sirve cuando algo se muestra **o no se muestra nada**. Si la izquierda es verdadera, se devuelve la derecha; si es falsa, se devuelve la izquierda:

```jsx
<ul>
  <li>Manzana</li>
  {isAdmin && <li>Panel de administración</li>}
  {age > 18 && <li>Vino</li>}
</ul>
```

**La trampa del `0`.** `false`, `null`, `undefined` y `true` no renderizan nada, pero el número `0` **sí se renderiza**. Con `&&`, si la izquierda es `0`, la expresión completa vale `0`:

```jsx
{messageCount && <p>Mensajes nuevos</p>}       {/* con 0 muestra "0" en pantalla */}
{messageCount > 0 && <p>Mensajes nuevos</p>}   {/* correcto: la izquierda es un booleano */}
```

La regla: la izquierda de `&&` debe ser un booleano, no un número.

### Listas con `.map` y `key`

Para crear una lista de elementos a partir de un array, se usa `.map()`, que devuelve un array nuevo. React sabe renderizar arrays de elementos JSX:

```jsx
const links = ['Inicio', 'Tienda', 'Contacto'];

<ul>
  {links.map((link) => (
    <li key={link}>{link}</li>
  ))}
</ul>
```

Cada elemento de una lista necesita un atributo **`key`**: un identificador **único entre sus hermanos** y **estable** entre renders. No es visible ni llega al componente como prop: React lo usa para saber qué elemento es cuál cuando la lista cambia (se agrega, se elimina o se reordena) y así conservar o actualizar el elemento correcto. Sin `key`, React muestra una advertencia y usa el índice como identificador.

Reglas:

* Debe ser único entre los elementos de **esa** lista (puede repetirse en otra lista distinta).
* Debe ser estable: no se genera durante el render (`Math.random()` haría que todos los elementos se recreen en cada render).
* Lo ideal es un identificador que venga de los datos (`item.id`).
* Si cada elemento renderiza varios nodos, se usa `<Fragment key={id}>`; la forma corta `<>...</>` no admite `key`.

-----

## Ejemplo completo

```jsx
const products = [
  { id: 1, name: 'Teclado', price: 45, stock: 3 },
  { id: 2, name: 'Mouse', price: 20, stock: 0 },
  { id: 3, name: 'Monitor', price: 180, stock: 8 },
];

export default function ProductList() {
  function handleBuy(id) {
    console.log('Comprar producto', id);
  }

  return (
    <section className="catalog">
      <h1>Catálogo</h1>
      {products.length > 0 ? (
        <ul>
          {products.map((product) => (
            <li key={product.id}>
              {product.name}: ${product.price}
              {product.stock === 0 && <span> (agotado)</span>}
              <button
                disabled={product.stock === 0}
                onClick={() => handleBuy(product.id)}
              >
                Comprar
              </button>
            </li>
          ))}
        </ul>
      ) : (
        <p>No hay productos.</p>
      )}
    </section>
  );
}
```

Aparecen todos los conceptos: `className`, llaves con expresiones, ternario para dos ramas, `&&` con un booleano (`=== 0`), evento con función flecha, y `.map` con `key` tomada de los datos.

-----

## Errores comunes

### 1. `class` en lugar de `className`

**Qué pasa:** React muestra una advertencia y el atributo no se comporta como se espera.
**Por qué:** en JSX los atributos usan los nombres de las propiedades del DOM.
**Arreglo:** escribir `className`.

### 2. Etiqueta sin cerrar

```jsx
<img src="foto.jpg" alt="Foto">   // error de sintaxis
```

**Por qué:** JSX exige que todas las etiquetas estén cerradas.
**Arreglo:** `<img src="foto.jpg" alt="Foto" />`.

### 3. Sentencia dentro de las llaves

`{if (x) { ... }}` o `{for (...) { ... }}` producen un error de sintaxis.
**Por qué:** las llaves aceptan expresiones, y una sentencia no produce un valor.
**Arreglo:** mover el `if` fuera del JSX, o usar ternario, `&&` o `.map`.

### 4. Llamar al manejador en lugar de pasarlo

```jsx
<button onClick={handleClick()}>Guardar</button>   // se ejecuta al renderizar
```

**Por qué:** `handleClick()` con paréntesis ejecuta la función en el render y pasa su resultado.
**Arreglo:** `onClick={handleClick}` o `onClick={() => handleClick(id)}`.

### 5. `&&` con un número a la izquierda

```jsx
{items.length && <List items={items} />}   // con 0 muestra "0"
```

**Por qué:** `0` es un valor renderizable y `&&` lo devuelve tal cual.
**Arreglo:** `{items.length > 0 && <List items={items} />}`.

### 6. Comillas en lugar de llaves en un atributo

`src="{url}"` pasa el texto literal `{url}`. Para pasar el valor de la variable se escribe `src={url}`.

### 7. Lista sin `key`, o con `key` inestable

**Qué pasa:** advertencia en consola; con `key` basado en índice o aleatorio, en listas que cambian pueden mezclarse estados (por ejemplo, el texto escrito en un input) o recrearse elementos.
**Arreglo:** usar un identificador único y estable de los datos.

-----

## En TypeScript

En un archivo `.tsx`, olvidar el cierre de una etiqueta es un error de sintaxis que el editor marca de inmediato.

**Eventos.** React exporta tipos por evento y por elemento. Si el manejador se declara aparte, se tipa así:

```tsx
function handleClick(event: React.MouseEvent<HTMLButtonElement>) {
  console.log(event.currentTarget.name);
}

function handleChange(event: React.ChangeEvent<HTMLInputElement>) {
  console.log(event.target.value);
}
```

Si el manejador se escribe en línea (`onClick={(e) => ...}`), TypeScript infiere el tipo de `e`. También existen tipos para la función completa, como `React.MouseEventHandler<HTMLButtonElement>`.

**Listas.** La prop `key` acepta `string | number` (`React.Key`, que también admite `null` y `undefined` en el tipo). Si el `id` del objeto es opcional (`id?: string`), conviene garantizar que exista antes de usarlo como `key`:

```tsx
type Product = { id: string; name: string };

function List({ products }: { products: Product[] }) {
  return (
    <ul>
      {products.map((p) => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

**El `0` con `&&`.** TypeScript no lo detecta: `items.length` es `number` y la expresión es válida. Es un problema de comportamiento en ejecución, no de tipos. La solución es la misma: comparar (`> 0`) o convertir a booleano.

-----

## Cuándo sí y cuándo no

* **`if` fuera del JSX:** hay varias ramas o la lógica es larga. Mantiene el JSX limpio.
* **Ternario:** dos alternativas visibles. Con ternarios anidados se pierde legibilidad; en ese caso conviene un `if` previo o un componente aparte.
* **`&&`:** algo se muestra o no se muestra. Asegúrate de que la izquierda sea booleana.
* **`.map`:** una lista de elementos a partir de un array. Usa como `key` un identificador de los datos. El índice solo es aceptable en listas estáticas que nunca se reordenan ni cambian.
* **Sin JSX:** es posible escribir React sin JSX llamando a `createElement`, pero rara vez conviene.

-----

## Resumen en 5 líneas

1. En JSX se usa `className`, todas las etiquetas se cierran (`<br />`) y los atributos van en camelCase.
2. Las llaves `{}` inyectan **expresiones** JavaScript como contenido o como valor de atributo; no aceptan sentencias.
3. Los eventos usan `onClick={handleClick}`: se pasa la función, no se llama.
4. Para condicionar: `if` fuera del JSX, ternario para dos ramas, `&&` para mostrar u omitir (con booleano a la izquierda, por el caso del `0`).
5. Las listas se generan con `.map` y cada elemento lleva una `key` única y estable, preferiblemente un `id` de los datos.

-----

## Para profundizar

<details>
<summary>Qué es JSX por debajo</summary>

JSX no es HTML: es sintaxis que un compilador transforma en llamadas a funciones de JavaScript que producen objetos que describen la UI (elementos React). Con el transform clásico, `<h1>Hola</h1>` se convierte en `React.createElement('h1', null, 'Hola')`; el transform moderno, usado por defecto en los proyectos actuales, genera llamadas a `jsx` importadas de `react/jsx-runtime`, sin necesidad de importar `React` en cada archivo. Ver [`createElement`](https://react.dev/reference/react/createElement) en la documentación de React.

</details>

<details>
<summary>Qué valores renderiza React y cuáles ignora</summary>

`true`, `false`, `null` y `undefined` son válidos como hijos pero no renderizan nada. Textos y números se renderizan, incluido `0`. Por eso `{cond && <A />}` es seguro con un booleano, y no lo es con un número.

</details>

<details>
<summary>Fragments</summary>

Un componente debe devolver un único elemento raíz. Para devolver varios sin añadir un nodo extra al DOM se usa `<>...</>` (Fragment). Si un Fragment necesita `key` (por ejemplo, dentro de un `.map`), se escribe `<Fragment key={id}>`, importado de `react`.

</details>

<details>
<summary>Por qué el índice es una mala `key` en listas que cambian</summary>

Si la clave es la posición, al insertar o eliminar un elemento todos los siguientes cambian de clave. React interpretará que "el elemento de la posición 2" sigue siendo el mismo y reutilizará su estado (por ejemplo, lo escrito en un input) para un dato distinto. Con un `id` propio, el estado sigue al dato.

</details>

-----

## En entrevista

### Respuesta corta (junior)

JSX permite escribir marcado dentro de JavaScript. Se usa `className` en vez de `class`, se cierran todas las etiquetas y, con llaves `{}`, se inserta JavaScript (variables, llamadas, ternarios). Las listas se renderizan con `.map` y cada elemento lleva una `key` única.

### Respuesta ampliada (semi-senior)

* **Compilación:** JSX se transforma en llamadas a funciones que crean elementos React; por eso hereda restricciones de JavaScript, como no poder usar `class`.
* **Llaves:** aceptan expresiones, no sentencias. Se usan como hijos o como valor de atributo; `{{ }}` es un objeto dentro de las llaves.
* **Condicionales:** `if` fuera del JSX para varias ramas, ternario para dos, `&&` para mostrar u omitir. Con `&&`, la izquierda debe ser booleana porque `0` sí se renderiza.
* **Eventos:** se pasa la función, no su resultado. Los eventos burbujean; se controlan con `stopPropagation` y `preventDefault`.
* **Listas:** `.map` con `key` única entre hermanos, estable y de los datos. React no pasa `key` al componente como prop.
* **TypeScript:** eventos tipados como `React.MouseEvent<HTMLButtonElement>` o `React.ChangeEvent<HTMLInputElement>`; el caso del `0` no lo detecta el compilador.

### Preguntas frecuentes de seguimiento

**1. ¿Por qué se usa `className` y no `class`?**
Porque JSX se compila a JavaScript, donde `class` es palabra reservada, y React usa los nombres de propiedades del DOM (`className`). En el HTML final se renderiza como `class`.

**2. ¿Por qué se necesita `key` y por qué no conviene usar el índice?**
React usa la `key` para identificar cada elemento entre renders y conservar su estado correctamente. Con el índice, si la lista se reordena o cambia, las claves apuntan a datos distintos y se pueden mezclar estados. Es mejor un `id` estable de los datos.

**3. ¿`&&` o ternario? ¿Qué es el bug del `0`?**
`&&` cuando algo se muestra o no; ternario cuando hay dos alternativas. Si la izquierda de `&&` es `0`, la expresión vale `0` y React lo renderiza; se evita con una comparación booleana (`count > 0 &&`).

**4. ¿Qué diferencia hay entre expresión y sentencia en JSX?**
Una expresión produce un valor (`a + b`, `cond ? x : y`) y cabe entre llaves. Una sentencia (`if`, `for`) ejecuta una acción sin producir un valor y no cabe; se coloca fuera del JSX.

**5. ¿Cómo se manejan los eventos?**
Se asigna una función al atributo `onEvento` en camelCase (`onClick={handleClick}`). Se pasa la función, no se llama; si necesita argumentos, se envuelve en una flecha (`() => remove(id)`).

**6. ¿Qué diferencia hay entre `onClick={handleClick}` y `onClick={handleClick()}`?**
El primero pasa la función para que React la ejecute al hacer clic. El segundo la ejecuta durante el render y pasa su resultado, que casi nunca es lo deseado.

-----

## Siguiente lección

Ya sabes inyectar JavaScript en JSX. El siguiente paso es ver cómo escribir JSX en varias líneas dentro de un componente: [JSX multilínea en un componente](04-Use%20Multiline%20JSX%20in%20a%20Component.md).
