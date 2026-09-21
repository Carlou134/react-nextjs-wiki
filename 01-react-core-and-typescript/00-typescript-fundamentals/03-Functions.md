# Functions: tipar parámetros y retorno

## En una frase

En TypeScript una función declara qué **tipos de argumentos** acepta y qué **tipo de valor** devuelve. El compilador verifica cada llamada contra esa firma antes de que el código se ejecute.

-----

## Antes de empezar

Conviene que ya sepas:

* Cómo se anotan variables y qué tipos primitivos existen: [Types](01-Types.md).
* Qué hace la opción `strict` del compilador: [Archivo tsconfig](02-Archivo%20tsconfig.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Parámetro:** variable que aparece en la declaración de la función. **Argumento:** valor concreto que se pasa al llamarla.
* **Firma (signature):** la parte de la función que describe sus parámetros y su tipo de retorno, sin el cuerpo.
* **Anotación de tipo:** `: tipo` escrito después de un nombre para declarar su tipo.
* **Inferencia:** el compilador deduce un tipo sin que lo escribas.
* **Callback:** función que se pasa como argumento a otra función para que esta la invoque.

-----

## El problema

JavaScript no valida los argumentos. Una función puede recibir un tipo inesperado y, aunque no lance error, producir un resultado equivocado:

```js
function printLengthOfText(text) {
  console.log(text.length);
}

printLengthOfText(3); // Imprime: undefined
```

La defensa clásica es validar en tiempo de ejecución:

```js
function printLengthOfText(text) {
  if (typeof text !== 'string') {
    throw new Error('El argumento no es una cadena de texto');
  }

  console.log(text.length);
}
```

Funciona, pero hay que repetir la validación en cada función y el error aparece cuando el código ya corre. TypeScript mueve esa verificación a **tiempo de compilación**: el error se marca en el editor, antes de ejecutar nada.

-----

## Cómo funciona

### Tipos de los parámetros

Se anota cada parámetro con `: tipo`, igual que una variable:

```ts
function greet(name: string) {
  console.log(`Hello, ${name}!`);
}

greet('Katz'); // Imprime: Hello, Katz!
greet(1337);   // Error: Argument of type 'number' is not assignable to parameter of type 'string'.
```

Un parámetro sin anotación (y sin valor por defecto) recibe el tipo implícito `any`, que desactiva la verificación. Con `strict` activado (que incluye `noImplicitAny`) eso es un **error de compilación**, así que en la práctica siempre se anotan.

```ts
function printKeyValue(key: string, value) {
  // Error con noImplicitAny: Parameter 'value' implicitly has an 'any' type.
  console.log(`${key}: ${value}`);
}
```

Una excepción: cuando la función se escribe en un lugar donde el tipo esperado ya es conocido (por ejemplo, un callback), los parámetros se infieren del contexto. Se ve más abajo.

### Parámetros opcionales

Por defecto, TypeScript exige un argumento por cada parámetro:

```ts
function greet(name: string) {
  console.log(`Hello, ${name || 'Anonymous'}!`);
}

greet(); // Error: Expected 1 arguments, but got 0.
```

Para que un parámetro pueda omitirse, agrega `?` después de su nombre. Su tipo pasa a ser `string | undefined`:

```ts
function greet(name?: string) {
  console.log(`Hello, ${name || 'Anonymous'}!`);
}

greet();        // Hello, Anonymous!
greet('Anders'); // Hello, Anders!
```

Los parámetros opcionales deben ir **después** de los obligatorios.

### Parámetros con valor por defecto

Si el parámetro tiene un valor por defecto, TypeScript infiere su tipo a partir de ese valor y el parámetro pasa a ser opcional sin necesidad de `?`:

```ts
function greet(name = 'Anonymous') {
  console.log(`Hello, ${name}!`);
}

greet();          // Hello, Anonymous!
greet('Ada');     // Hello, Ada!
greet(undefined); // Hello, Anonymous!  (undefined activa el valor por defecto)
greet(42);        // Error: 'number' no es asignable a 'string'
```

Dentro de la función, `name` es `string` (no `string | undefined`), por lo que no hace falta el `|| 'Anonymous'`. Además, el valor por defecto solo se aplica con `undefined`, mientras que `||` reemplazaría también `''` y `0`.

### Parámetros rest

Un parámetro rest (`...`) recoge los argumentos restantes en un array. Se tipa como array y debe ser el último:

```ts
function sum(label: string, ...numbers: number[]): string {
  const total = numbers.reduce((acc, n) => acc + n, 0);
  return `${label}: ${total}`;
}

sum('Total', 1, 2, 3); // "Total: 6"
```

### Tipo de retorno inferido

TypeScript deduce el tipo de retorno a partir de las sentencias `return`:

```ts
function createGreeting(name: string) {
  return `Hello, ${name}!`; // retorno inferido: string
}

const myGreeting = createGreeting('Aisle Nevertell'); // string
```

La inferencia detecta incoherencias en el sitio de uso:

```ts
function ouncesToCups(ounces: number) {
  return `${ounces / 16} cups`;
}

const liquidAmount: number = ouncesToCups(3);
// Error: Type 'string' is not assignable to type 'number'.
```

### Tipo de retorno explícito

Se escribe `: tipo` después del paréntesis de cierre de los parámetros. Con la anotación, TypeScript revisa **cada `return` dentro de la función**, no solo el sitio de uso:

```ts
function createGreeting(name?: string): string {
  if (name) {
    return `Hello, ${name}!`;
  }

  return undefined;
  // Error (con strictNullChecks): Type 'undefined' is not assignable to type 'string'.
}

const createArrowGreeting = (name?: string): string => {
  return name ? `Hello, ${name}!` : 'Hello!';
};
```

Sin la anotación, el error se manifestaría más lejos, en quien consume la función. Anotar el retorno de las funciones que exportas fija el **contrato** de forma explícita: un cambio accidental en el cuerpo no altera en silencio el tipo que ven los demás módulos.

### `void` y `never`

Son dos tipos de retorno distintos:

* **`void`:** la función termina, pero no devuelve un valor útil.
* **`never`:** la función **nunca termina normalmente**: siempre lanza una excepción o entra en un bucle infinito.

```ts
function logGreeting(name: string): void {
  console.log(`Hello, ${name}!`);
}

function fail(message: string): never {
  throw new Error(message);
}
```

### Funciones como tipo

Una función también es un valor y tiene tipo. Se describe con una sintaxis parecida a una función flecha: parámetros entre paréntesis, `=>` y el tipo de retorno.

```ts
type Operation = (a: number, b: number) => number;

const add: Operation = (a, b) => a + b;       // a y b se infieren como number
const multiply: Operation = (a, b) => a * b;
```

Los nombres de los parámetros en el tipo son solo documentación: la implementación puede usar otros. Lo que se compara son las posiciones y los tipos.

### Callbacks

El tipo de una función sirve para tipar parámetros que reciben callbacks:

```ts
function fetchUser(id: number, onSuccess: (name: string) => void): void {
  // simulación: en la práctica vendría de una petición
  onSuccess(`User ${id}`);
}

fetchUser(1, (name) => console.log(name.toUpperCase())); // name se infiere como string
```

Dos reglas de compatibilidad útiles:

* Un callback puede declarar **menos** parámetros de los que el llamador le pasa (`[1, 2].forEach(() => {})` es válido), pero no más.
* Un tipo `() => void` acepta funciones que sí devuelven un valor; el resultado simplemente se ignora. Por eso `items.forEach((x) => out.push(x))` compila aunque `push` devuelva un número. Esto solo aplica a **tipos de función**: una declaración `function f(): void { return 1; }` sí es un error.

### Sobrecargas

Una función puede tener varias firmas de llamada. Se escriben las firmas públicas (sin cuerpo) y, debajo, una única **implementación** compatible con todas:

```ts
function format(value: string): string;
function format(value: number, decimals: number): string;
function format(value: string | number, decimals?: number): string {
  if (typeof value === 'string') {
    return value.trim();
  }
  return value.toFixed(decimals);
}

format('  hola ');  // "hola"
format(3.14159, 2); // "3.14"
format(3.14159);    // Error: ninguna sobrecarga acepta 1 argumento de tipo number
```

La firma de la implementación **no es visible** desde afuera: solo cuentan las sobrecargas declaradas. Úsalas cuando el tipo de retorno o la combinación de parámetros dependan de cómo se llama la función; si basta un tipo unión o un parámetro opcional, es más simple no sobrecargar.

### Documentar con comentarios TSDoc

Un comentario que empieza con `/**` y va justo encima de la función es un **comentario de documentación**. TypeScript lo interpreta como JSDoc y el editor lo muestra al pasar el cursor sobre la función. `@param` describe cada parámetro y `@returns` el resultado; el formato `@param x - texto` es la convención de TSDoc:

```ts
/**
 * Devuelve la suma de dos números.
 *
 * @param x - El primer número de entrada
 * @param y - El segundo número de entrada
 * @returns La suma de `x` y `y`
 */
function getSum(x: number, y: number): number {
  return x + y;
}
```

-----

## Ejemplo completo

```ts
type Discount = (price: number) => number;

/**
 * Calcula el total de un pedido aplicando descuentos opcionales.
 *
 * @param prices - Precios unitarios de los productos
 * @param taxRate - Impuesto como fracción (por defecto 0.19)
 * @param discounts - Funciones que se aplican en orden al subtotal
 */
function calculateTotal(
  prices: number[],
  taxRate = 0.19,
  ...discounts: Discount[]
): number {
  const subtotal = prices.reduce((acc, p) => acc + p, 0);
  const discounted = discounts.reduce((acc, apply) => apply(acc), subtotal);
  return discounted * (1 + taxRate);
}

const tenPercentOff: Discount = (price) => price * 0.9;

calculateTotal([100, 50]);                     // 178.5
calculateTotal([100, 50], 0.21, tenPercentOff); // 163.35
calculateTotal([100, 50], 'alto');             // Error: 'string' no es asignable a 'number'
```

Puntos clave:

1. `taxRate` tiene valor por defecto: es opcional y su tipo se infiere como `number`.
2. `...discounts` recoge cero o más funciones; el tipo `Discount` describe cada una.
3. El retorno `: number` está anotado, así que cualquier `return` incompatible se marca dentro de la función.
4. En `discounts.reduce(...)`, `apply` se infiere como `Discount` por el contexto.

-----

## Errores comunes

### 1. Dejar parámetros sin tipo

```ts
function double(n) { return n * 2; } // Error con noImplicitAny
```

**Por qué pasa:** sin anotación el parámetro sería `any` y no habría verificación.
**Solución:** anota el parámetro: `(n: number)`.

### 2. Usar `?` y esperar que el valor sea `string`

```ts
function upper(text?: string) {
  return text.toUpperCase(); // Error: 'text' is possibly 'undefined'
}
```

**Por qué pasa:** un parámetro opcional es `string | undefined`.
**Solución:** comprueba antes de usarlo (`if (text) ...`, `text?.toUpperCase()`), o usa un valor por defecto (`text = ''`).

### 3. Poner un parámetro obligatorio después de uno opcional

```ts
function f(a?: number, b: number) {} // Error: A required parameter cannot follow an optional parameter.
```

**Solución:** reordena los parámetros, o declara `a: number | undefined` si necesitas que exista la posición.

### 4. Confundir `void` con `undefined`

Una función con retorno `void` no debe usarse para obtener un valor. Además, una función con retorno explícito `string` no puede terminar sin `return` en todos los caminos: el compilador lo marca con "Function lacks ending return statement".
**Solución:** devuelve un valor en todas las ramas, o cambia el retorno a `string | undefined`.

### 5. Sobrecargar cuando bastaba una unión

Escribir muchas firmas repetidas para casos que un tipo unión o un parámetro opcional resuelve. **Solución:** usa `value: string | number` y estrecha el tipo dentro del cuerpo; reserva las sobrecargas para cuando el retorno cambia según los argumentos.

-----

## Cuándo sí y cuándo no

**Anota siempre los parámetros.** Con `strict`, no anotarlos es un error salvo que el contexto los infiera (callbacks).

**Anota el retorno explícitamente** en funciones exportadas, en APIs públicas y cuando quieras que el compilador verifique el cuerpo contra un contrato. Es más útil aún en funciones asíncronas: `async function load(): Promise<User>` (el retorno de una función `async` siempre es una `Promise`).

**Deja que se infiera el retorno** en funciones locales pequeñas y callbacks cortos, donde el tipo es evidente y anotarlo solo añade ruido.

**Prefiere un valor por defecto a `?`** cuando exista un valor razonable: simplifica el cuerpo, porque el parámetro ya no es `undefined`.

-----

## Resumen en 5 líneas

1. Los parámetros se anotan con `: tipo`; sin anotación son `any` implícito, un error con `strict`.
2. `?` hace un parámetro opcional (`T | undefined`); un valor por defecto lo hace opcional e infiere su tipo.
3. El tipo de retorno se infiere del `return`, pero anotarlo explícitamente verifica el cuerpo y fija el contrato.
4. `void` indica que no hay valor de retorno útil; `never`, que la función no termina normalmente.
5. Las funciones tienen tipo (`(a: number) => string`), y con él se tipan callbacks y variables; las sobrecargas cubren firmas alternativas.

-----

## Para profundizar

<details>
<summary>Por qué un tipo `() => void` acepta funciones que devuelven valores</summary>

Un tipo función con retorno `void` significa "el llamador no usará el resultado", no "la función debe devolver `undefined`". Por eso esto compila:

```ts
type Callback = () => void;

const cb: Callback = () => 42; // válido
const result = cb();           // result es void: no se puede usar como número
```

Esta regla permite pasar funciones existentes como callbacks sin envolverlas. No aplica a la declaración directa de una función con retorno `void` que devuelve un valor: ahí sí hay error.

</details>

<details>
<summary>`this` como parámetro falso</summary>

En una función se puede declarar el tipo de `this` como primer "parámetro". No existe en tiempo de ejecución; solo lo usa el compilador:

```ts
type User = { name: string };

function sayName(this: User): string {
  return this.name;
}

sayName.call({ name: 'Ana' }); // válido
sayName();                     // Error: The 'this' context of type 'void' is not assignable to method's 'this' of type 'User'.
```

Las funciones flecha no tienen `this` propio: lo toman del ámbito donde se definen.

</details>

<details>
<summary>Tipos de función con firmas de llamada</summary>

Además de la sintaxis flecha, se puede describir una función con una firma de llamada dentro de un objeto. Es útil cuando la función tiene además propiedades:

```ts
type Counter = {
  (): number;
  reset: () => void;
};
```

Para la mayoría de los casos, `(a: number) => string` es suficiente y más legible.

</details>

-----

## En entrevista

### Respuesta corta (junior)

En TypeScript se anotan los parámetros y el tipo de retorno de una función para que el compilador verifique cada llamada. Un parámetro opcional se marca con `?` o se le da un valor por defecto. Si no se anota el retorno, TypeScript lo infiere del `return`. `void` se usa cuando la función no devuelve nada.

### Respuesta ampliada (semi-senior)

* **Parámetros:** sin anotación son `any` implícito, un error bajo `noImplicitAny` (parte de `strict`), salvo que el contexto (por ejemplo, un callback) permita inferirlos.
* **Opcionales:** `x?: T` equivale a `T | undefined` y debe ir después de los obligatorios. Un valor por defecto lo hace opcional y estrecha el tipo interno a `T`.
* **Retorno:** inferido o explícito. Anotarlo en funciones exportadas fija el contrato y ubica el error dentro de la función en vez de en el consumidor.
* **`void` vs `never`:** `void` es "sin valor útil"; `never` es "no termina normalmente" (lanza o no acaba nunca).
* **Tipos de función:** `type Fn = (a: A) => B`. Los nombres de parámetros no importan, solo posiciones y tipos. Un callback puede declarar menos parámetros de los que recibe.
* **Regla del `void` en tipos función:** un `() => void` acepta funciones que devuelven valores; el resultado se ignora.
* **Sobrecargas:** varias firmas públicas y una implementación compatible, que no es visible desde afuera. Se prefieren uniones o genéricos cuando bastan.
* **Async:** una función `async` devuelve siempre `Promise<T>`.

### Preguntas frecuentes de seguimiento

**1. ¿Cuál es la diferencia entre `name?: string` y `name = 'x'`?**
Ambos hacen el parámetro opcional. Con `?`, dentro de la función el tipo es `string | undefined` y hay que manejarlo. Con valor por defecto, el tipo interno es `string` y `undefined` activa el valor por defecto.

**2. ¿Qué diferencia hay entre `void` y `never`?**
`void` es el retorno de una función que termina sin valor útil (como `console.log`). `never` es el de una que nunca llega a devolver: lanza una excepción o entra en un bucle infinito.

**3. ¿Conviene anotar siempre el tipo de retorno?**
En funciones exportadas y APIs públicas, sí: fija el contrato y da errores más cercanos a la causa. En funciones locales o callbacks cortos, la inferencia suele bastar.

**4. ¿Por qué `[1, 2].forEach((n) => out.push(n))` compila si `push` devuelve un número?**
Porque el callback de `forEach` está tipado como `(value) => void`, y un tipo función con retorno `void` acepta funciones que devuelven cualquier valor: se ignora.

**5. ¿Cuándo usar sobrecargas y cuándo un tipo unión?**
Si la función se comporta igual con distintos tipos de entrada, basta una unión. Las sobrecargas se justifican cuando el tipo de retorno o la combinación válida de parámetros cambia según la llamada.

**6. ¿Qué tipo tiene el retorno de una función `async`?**
Siempre `Promise<T>`. Si anotas el retorno, debe ser `Promise<T>`, no `T`.

-----

## Siguiente lección

Ya sabes tipar funciones. Ahora veremos cómo tipar colecciones de valores: [Arrays](04-Arrays.md).
