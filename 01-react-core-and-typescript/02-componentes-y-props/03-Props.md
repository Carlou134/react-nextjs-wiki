# Props: pasar datos entre componentes

## En una frase

Las **props** son el objeto con los datos que un componente padre entrega a un hijo al renderizarlo; son de solo lectura y viajan en una sola dirección, de arriba hacia abajo.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un componente de función y cómo devuelve JSX: [Tu primer componente](01-Your%20First%20React%20Component.md).
* Cómo un componente renderiza a otros: [Componentes que renderizan componentes](02-Components%20Render%20Other%20Components.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Props:** objeto que React construye con los atributos escritos en la etiqueta del componente y entrega como primer parámetro de la función.
* **Componente padre / hijo:** el padre es el que escribe `<Hijo />` en su JSX; el hijo es el que se renderiza.
* **Manejador de eventos (event handler):** función que se ejecuta como respuesta a un evento, como un clic.
* **Callback:** función que se pasa a otra para que esta la llame cuando corresponda.

-----

## El problema

Un componente que muestra siempre el mismo contenido solo sirve una vez. Para reutilizarlo hay que poder configurarlo desde afuera:

```jsx
function GreetingJamel() { return <h1>Hola, Jamel</h1>; }
function GreetingEsmeralda() { return <h1>Hola, Esmeralda</h1>; }
```

Copiar el componente por cada nombre no escala. Las props resuelven esto: el componente se escribe una vez y el padre le entrega los datos que cambian. Funcionan como los argumentos de una función.

-----

## Cómo funciona

### Pasar props

Se pasan como atributos en la etiqueta del componente. Los textos van entre comillas; cualquier otro valor (número, booleano, arreglo, objeto, función) va entre llaves:

```jsx
<Greeting name="The Queen Mary" age={56} haunted={true} tags={["barco", "museo"]} />
```

El nombre del atributo lo eliges tú.

### Recibir props

React reúne todos los atributos en un único objeto y lo pasa como primer parámetro:

```jsx
function Button(props) {
  return <button>{props.displayText}</button>;
}
```

Es equivalente a recibir `{ displayText: "..." }` y leer su propiedad con notación de punto. Si no se pasa ninguna prop, `props` es un objeto vacío, no `undefined`.

### Desestructurar

Como `props` es un objeto, se puede desestructurar en el parámetro. Es la forma más habitual porque deja visibles las props que usa el componente:

```jsx
function Button({ displayText }) {
  return <button>{displayText}</button>;
}
```

Ambas formas son equivalentes.

### Pasar props de componente a componente

Es el uso más común: un padre entrega datos a un hijo.

```jsx
function App() {
  return <Product name="Apple Watch" price={399} rating="4.5/5.0" />;
}
```

`App` es el padre y `Product` el hijo. Las props viajan en una sola dirección (**flujo unidireccional**): del padre al hijo, nunca al revés. Si el hijo necesita comunicar algo al padre, el padre le pasa una función (ver más abajo).

Las props son **de solo lectura**: cada render recibe un objeto nuevo y el componente no debe modificarlo. Si un componente necesita valores distintos, el padre le pasa nuevas props en su siguiente render. Si el dato debe cambiar dentro del propio componente, ese dato es **estado**, no una prop.

Para reenviar todas las props a otro componente existe la sintaxis spread (`<Avatar {...props} />`). Úsala con moderación: oculta qué datos se pasan realmente.

### Renderizar según las props

Una prop no solo se muestra: también sirve para decidir qué mostrar.

```jsx
function LoginMsg({ isValid }) {
  if (isValid) {
    return <h2>Inicio de sesión exitoso.</h2>;
  }
  return <h2>Falló el inicio de sesión.</h2>;
}
```

Aquí la prop no se imprime; determina qué JSX devuelve el componente. (En el ejemplo se recibe un booleano ya calculado por el padre; nunca compares ni muestres contraseñas reales en un componente.)

### Manejadores de eventos como props

Una función también es un valor y se puede pasar como prop. El patrón habitual: el padre define **qué** ocurre y el hijo decide **cuándo** se dispara.

```jsx
function Talker() {
  function talk() {
    alert('blah blah blah');
  }

  return <Button talk={talk} />;
}

function Button({ talk }) {
  return <button onClick={talk}>Hablar</button>;
}
```

Se pasa la **función**, sin paréntesis. `talk={talk()}` la ejecutaría durante el render de `Talker` y pasaría su resultado (`undefined`) como prop. React es quien llama a `talk` cuando ocurre el clic.

Secuencia: React renderiza `Talker`, este crea `talk` y se la pasa a `Button`; cuando el usuario hace clic, React invoca la función.

### Convención `handleX` y `onX`

Hay dos nombres que elegir, ambos en el padre:

* El manejador: `handle` + evento (`handleClick`, `handleHover`).
* La prop que lo transporta: `on` + evento (`onClick`, `onHover`).

```jsx
function MyComponent() {
  function handleHover() {
    console.log('hover');
  }

  return <Child onHover={handleHover} />;
}
```

`onClick` no es un evento mágico en todos lados. Sobre un elemento nativo (`<button>`, `<div>`), React registra un evento real del DOM. Sobre un componente propio (`<Button onClick={...} />`), es solo el nombre de una prop; el evento existe cuando, dentro del componente, esa prop se asigna a un elemento nativo:

```jsx
function Button({ onClick }) {
  return <button onClick={onClick}>Click me</button>;
}
```

Por eso, `onX` en componentes propios es una convención de nombres, no un mecanismo del lenguaje.

### `props.children`

Todo lo escrito entre la etiqueta de apertura y la de cierre de un componente llega en la prop `children`:

```jsx
function BigButton({ children }) {
  return <button>{children}</button>;
}

<BigButton>Texto</BigButton>          // children es el string "Texto"
<BigButton><LilButton /></BigButton>  // children es el elemento <LilButton />
<BigButton />                         // children es undefined
```

Con varios hijos, `children` es un arreglo; con uno solo, es ese valor sin envolver. Sirve para componentes contenedor (layouts, tarjetas, paneles) cuyo contenido cambia por completo según quien los use. Es la base de la composición.

### Valores por defecto

Si una prop puede omitirse, define un valor por defecto al desestructurar:

```jsx
function Example({ text = 'Texto por defecto' }) {
  return <h1>{text}</h1>;
}
```

También puedes hacerlo en el cuerpo: `const { text = 'Texto por defecto' } = props;`.

El valor por defecto se usa solo si la prop falta o vale `undefined`. Con `null`, `0` o `''` **no** se aplica.

`Componente.defaultProps` en componentes de función fue eliminado en React 19; los defaults al desestructurar son la alternativa oficial. Los componentes de clase lo conservan.

-----

## Ejemplo completo

```tsx
import type { ReactNode } from 'react';

type CardProps = {
  title: string;
  price?: number;
  onBuy: (title: string) => void;
  children?: ReactNode;
};

function Card({ title, price = 0, onBuy, children }: CardProps) {
  return (
    <article>
      <h2>{title}</h2>
      {price === 0 ? <p>Gratis</p> : <p>${price}</p>}
      {children}
      <button onClick={() => onBuy(title)}>Comprar</button>
    </article>
  );
}

const products = [
  { id: 1, title: 'Apple Watch', price: 399 },
  { id: 2, title: 'Guía de React', price: 0 },
];

export default function Shop() {
  function handleBuy(title: string) {
    console.log(`Compraste: ${title}`);
  }

  return (
    <section>
      {products.map((p) => (
        <Card key={p.id} title={p.title} price={p.price} onBuy={handleBuy}>
          <small>Envío incluido</small>
        </Card>
      ))}
    </section>
  );
}
```

Puntos clave:

1. `Shop` (padre) define `handleBuy` y lo pasa como `onBuy`; `Card` decide cuándo llamarlo.
2. `price` es opcional y tiene valor por defecto; `children` se renderiza donde `Card` lo indica.
3. Se pasa `key` en el `map`; React la usa para identificar cada elemento y no llega a `Card` como prop.
4. `onClick={() => onBuy(title)}` usa una función flecha porque hay que pasar un argumento.

-----

## Errores comunes

### 1. Mutar las props

```jsx
function Badge(props) {
  props.label = props.label.toUpperCase();   // error
  return <span>{props.label}</span>;
}
```

**Por qué pasa:** parece una variable local. **Qué ocurre:** las props son de solo lectura; modificarlas produce comportamiento impredecible (en modo estricto, el objeto está congelado y falla). **Solución:** deriva un valor nuevo durante el render.

```jsx
function Badge({ label }) {
  const upper = label.toUpperCase();
  return <span>{upper}</span>;
}
```

### 2. Ejecutar la función en lugar de pasarla

```jsx
<button onClick={handleClick()}>Enviar</button>   // se ejecuta al renderizar
```

**Por qué pasa:** con paréntesis la función se invoca durante el render y `onClick` recibe su resultado. **Solución:** pasa la referencia (`onClick={handleClick}`) o una función que la llame (`onClick={() => handleClick(id)}`).

### 3. Olvidar `key` al renderizar listas

```jsx
{products.map((p) => <Card title={p.title} />)}   // advertencia de key
```

**Por qué pasa:** React necesita identificar cada elemento entre renders. **Solución:** usa un identificador estable (`key={p.id}`). Evita el índice si la lista se reordena o se borran elementos. Recuerda que `key` no se lee como `props.key`; si el hijo necesita el valor, pásalo con otro nombre.

### 4. Guardar en estado una copia de una prop

```jsx
const [name, setName] = useState(props.name);   // no se actualiza si la prop cambia
```

**Por qué pasa:** `useState` usa el valor inicial solo en el primer render. **Solución:** usa la prop directamente. Copiarla solo tiene sentido si es un valor inicial editable y se nombra así (`initialName`).

### 5. Esperar que `children` siempre exista

`children` es `undefined` cuando el componente se usa autocerrado. **Solución:** decide si es obligatorio o tipa como opcional y maneja el caso vacío.

-----

## En TypeScript

### Tipar las props

Declara la forma de las props con `type` o `interface` y anota el parámetro desestructurado:

```tsx
type ButtonProps = {
  displayText: string;
};

function Button({ displayText }: ButtonProps) {
  return <button>{displayText}</button>;
}
```

TypeScript avisa en compilación si falta una prop obligatoria, si el tipo es incorrecto o si escribes mal un nombre. Para props, `type` e `interface` son intercambiables; elige uno y sé consistente. Más detalle en [Tipado de Props y Funciones](../11-typescript-y-react/01-Tipado%20de%20Props%20y%20Funciones.md).

### Props opcionales y valores por defecto

Una prop con valor por defecto se marca con `?`; sin él, TypeScript la exige aunque el default exista:

```tsx
type ExampleProps = {
  text?: string;
};

function Example({ text = 'Texto por defecto' }: ExampleProps) {
  return <h1>{text}</h1>;
}
```

### `children`

Se tipa con `React.ReactNode`, que cubre todo lo renderizable (texto, números, elementos, arreglos, `null`, `undefined`). Si el componente puede usarse sin hijos, márcalo opcional:

```tsx
import type { ReactNode } from 'react';

type BigButtonProps = {
  children?: ReactNode;
};

function BigButton({ children }: BigButtonProps) {
  return <button>{children}</button>;
}
```

Alternativa: `PropsWithChildren<Props>`, que agrega `children?: ReactNode` al tipo.

### Callbacks y eventos

Tipa las funciones con su firma completa:

```tsx
type ButtonProps = {
  talk: () => void;                            // sin argumentos
  onBuy: (id: number) => void;                 // con argumento
  onClick?: React.MouseEventHandler<HTMLButtonElement>;   // evento del DOM
};
```

Para pasar un manejador a un elemento nativo, usa el tipo del evento (`React.MouseEvent<HTMLButtonElement>`, `React.ChangeEvent<HTMLInputElement>`). Si un callback exige un argumento y lo llamas sin él, TypeScript lo marca.

### Sobre `React.FC`

No es necesario: anotar el parámetro (`function Button(props: ButtonProps)`) es más simple, deja los genéricos y el valor de retorno inferidos y evita depender de un tipo extra. Es una preferencia de estilo habitual, no una regla de React.

-----

## Cuándo sí y cuándo no

Guía de uso de cada concepto:

* **Pasar props:** cuando el padre necesita configurar al hijo.
* **Renderizado según props:** cuando el hijo decide qué mostrar según lo que recibió.
* **Evento como prop:** cuando el padre debe reaccionar a algo que ocurre dentro del hijo.
* **`handleX` / `onX`:** siempre que definas un manejador y lo pases como prop, para que se entienda sin rastrear la implementación.
* **`children`:** cuando el contenido interno varía tanto que enumerarlo como props nombradas no tiene sentido.
* **Valores por defecto:** cuando una prop es opcional y existe un valor razonable si nadie la pasa.

Props, estado y Context:

* **Props:** datos que llegan de un ancestro y el componente solo lee. Es la opción por defecto.
* **Estado:** datos que el propio componente controla y cambian con el tiempo (ver [useState](../04-hooks-y-context/01-The%20State%20Hook.md)).
* **Context:** datos que muchos componentes a distintos niveles necesitan (tema, usuario), cuando pasarlos nivel por nivel (**prop drilling**) se vuelve incómodo (ver [React Context](../04-hooks-y-context/04-React%20Context.md)).

Antes de usar Context, prueba la composición: pasar `children` o componentes ya armados evita atravesar niveles intermedios sin necesidad.

-----

## Resumen en 5 líneas

1. Las props son un objeto de solo lectura que el padre pasa al hijo con atributos JSX.
2. Se reciben como parámetro y suelen desestructurarse: `function Button({ text })`.
3. Las funciones también son props; se pasan sin paréntesis y se nombran `handleX` (manejador) y `onX` (prop).
4. `children` recibe el contenido entre las etiquetas; los valores por defecto se ponen al desestructurar.
5. Con TypeScript, declara un `type` de props; usa `?` para las opcionales y `ReactNode` para `children`.

-----

## Para profundizar

<details>
<summary>Por qué las props son inmutables</summary>

React asume que un componente es una función pura respecto a sus props y su estado: mismas entradas, mismo JSX. Si mutaras las props, el padre y otros componentes que comparten ese objeto verían cambios sin que React lo sepa, y no habría re-render. Cada render recibe un objeto de props nuevo; para cambiar datos, se usa estado en el dueño del dato.

</details>

<details>
<summary>Prop drilling y cómo evitarlo</summary>

Ocurre cuando una prop atraviesa varios componentes intermedios que no la usan, solo para llegar a uno más profundo. Opciones: (1) composición con `children` o pasando elementos ya construidos, (2) Context para datos globales o de amplio alcance, (3) acercar el estado al componente que lo usa. No conviene saltar a Context al primer nivel de anidación: dos o tres niveles de props explícitas son legibles y fáciles de rastrear.

</details>

<details>
<summary>`ref` como prop en React 19</summary>

Desde React 19, los componentes de función pueden recibir `ref` como una prop más, por lo que `forwardRef` ya no es necesario en componentes nuevos. Las refs pasadas a componentes de clase siguen apuntando a la instancia y no llegan como prop.

</details>

-----

## En entrevista

### Respuesta corta (junior)

Las props son los datos que un componente padre pasa a un hijo mediante atributos JSX. El hijo las recibe como un objeto de solo lectura, normalmente desestructurado. Permiten reutilizar un componente con distintos datos y también pasar funciones para que el hijo avise al padre.

### Respuesta ampliada (semi-senior)

* **Flujo unidireccional:** los datos bajan del padre al hijo; el hijo se comunica hacia arriba llamando callbacks recibidos como props.
* **Solo lectura:** cada render recibe un objeto nuevo; el componente no debe mutarlo. Lo que cambia dentro del componente es estado.
* **Funciones como props:** se pasa la referencia, no el resultado. La convención es `handleX` para el manejador y `onX` para la prop.
* **`children` y composición:** permiten componentes contenedor y evitan prop drilling sin recurrir a Context.
* **Valores por defecto:** al desestructurar; solo aplican con `undefined`. `defaultProps` en componentes de función se eliminó en React 19.
* **Tipado:** `type` o `interface` para las props, `?` para opcionales, `ReactNode` para `children` y firmas explícitas para callbacks.
* **Errores típicos:** mutar props, `onClick={fn()}`, olvidar `key`, copiar props a estado.

### Preguntas frecuentes de seguimiento

**1. ¿Cuál es la diferencia entre props y estado?**
Las props llegan desde el padre y son de solo lectura; el estado pertenece al componente, lo conserva React entre renders y se cambia con su setter. Cambiar cualquiera de los dos provoca un nuevo render.

**2. ¿Por qué se dice que el flujo de datos es unidireccional?**
Porque los datos solo bajan del padre al hijo. Para que el hijo influya en el padre, este le pasa una función y el hijo la invoca.

**3. ¿Se pueden mutar las props?**
No. Son de solo lectura; hacerlo rompe el modelo de React y no provoca re-render. Deriva un valor nuevo o, si el dato debe cambiar, elévalo a estado del padre.

**4. ¿Qué es el prop drilling y cómo se evita?**
Pasar una prop por varios niveles intermedios que no la usan. Se reduce con composición (`children`), acercando el estado a quien lo usa o con Context si el dato es de amplio alcance.

**5. ¿Para qué sirve `children`?**
Recibe el contenido entre las etiquetas del componente. Permite crear contenedores reutilizables (layouts, tarjetas) sin conocer de antemano su contenido.

**6. ¿Cómo se definen valores por defecto en React 19?**
Al desestructurar: `function Card({ size = 'md' })`. Solo se aplican si la prop falta o es `undefined`. `defaultProps` en componentes de función ya no se soporta.

-----

## Siguiente lección

Con componentes y props dominados, sigue el orden de carpetas: primero el tooling ([Creating a React App](../03-tooling-y-devtools/01-Creating%20a%20React%20App.md)) y después los Hooks, empezando por [useState](../04-hooks-y-context/01-The%20State%20Hook.md).
