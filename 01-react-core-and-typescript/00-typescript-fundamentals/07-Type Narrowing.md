# Type Narrowing: estrechar un tipo con comprobaciones

## En una frase

El **narrowing** (estrechamiento) es el análisis con el que TypeScript, a partir de comprobaciones en tu código (`typeof`, `instanceof`, `in`, igualdad, etc.), reduce un tipo amplio (por ejemplo una unión) a uno más específico dentro de cada rama.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un tipo unión (`string | number`): [Union Types](06-Union%20Types.md).
* Cómo se definen tipos de objeto propios: [Custom Types](05-Custom%20Types.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Narrowing:** reducción de un tipo a un subconjunto más específico según el flujo del código.
* **Type guard (guarda de tipo):** expresión o función cuya comprobación en tiempo de ejecución permite a TypeScript hacer narrowing.
* **Discriminante:** propiedad con tipo literal, distinta en cada miembro de una unión, que identifica a qué miembro corresponde un valor.
* **Análisis de flujo de control:** el mecanismo del compilador que sigue `if`, `return`, `switch` y demás para saber qué tipo tiene una variable en cada punto.

-----

## El problema

Con una unión, TypeScript solo permite lo que es válido para **todos** los miembros:

```ts
function shout(value: string | number) {
  return value.toUpperCase();
  // Error: Property 'toUpperCase' does not exist on type 'string | number'.
  //        Property 'toUpperCase' does not exist on type 'number'.
}
```

Los tipos se borran al compilar, así que TypeScript no puede saber en qué caso estás. Hace falta una comprobación que exista **en tiempo de ejecución** y que el compilador sepa interpretar.

-----

## Cómo funciona

TypeScript analiza el flujo del código. Dentro de cada rama, el tipo de la variable es el que queda después de aplicar las comprobaciones anteriores.

```
value: string | number | null
   |
   |-- if (value === null) return      -> value: string | number
   |
   |-- if (typeof value === "string")  -> value: string
   |   (rama else)                     -> value: number
```

### `typeof`

Comprueba el tipo primitivo. TypeScript entiende los resultados `"string"`, `"number"`, `"bigint"`, `"boolean"`, `"symbol"`, `"undefined"`, `"object"` y `"function"`.

```ts
function formatId(id: string | number): string {
  if (typeof id === "string") {
    return id.toUpperCase(); // id: string
  }
  return id.toFixed(0);      // id: number
}
```

`typeof null` devuelve `"object"` (rasgo histórico de JavaScript). Por eso `typeof x === "object"` deja `null` dentro del tipo estrechado.

### `else` y salida temprana (`return`, `throw`)

El `else` recibe el tipo complementario. Además, si la rama del `if` termina la ejecución (`return`, `throw`, `continue`), el código posterior ya no puede ser ese caso:

```ts
type Tea = { steep(): string };
type Coffee = { pourOver(): string };

function brew(drink: Tea | Coffee): string {
  if ("steep" in drink) {
    return drink.steep();   // drink: Tea
  }
  return drink.pourOver();  // drink: Coffee
}
```

### `instanceof`

Comprueba si un valor fue creado por un constructor (clase). Sirve para tipos que existen como clase en tiempo de ejecución, como `Date`, `Error` o clases propias.

```ts
function describeError(error: unknown): string {
  if (error instanceof Error) {
    return error.message;        // error: Error
  }
  if (typeof error === "string") {
    return error;                // error: string
  }
  return "Error desconocido";
}
```

Un `type` o `interface` no existe en tiempo de ejecución, por lo que no se puede usar con `instanceof`.

### `in`

`"prop" in obj` comprueba si la propiedad existe en el objeto o en su cadena de prototipos. Con uniones de objetos, TypeScript conserva los miembros que tienen (o pueden tener) esa propiedad.

```ts
type Tennis = { serve(): void };
type Soccer = { kick(): void };

function play(sport: Tennis | Soccer): void {
  if ("serve" in sport) {
    sport.serve(); // sport: Tennis
  } else {
    sport.kick();  // sport: Soccer
  }
}
```

### Igualdad

`===`, `!==`, `==` y `!=` estrechan cuando se compara con literales o con otro valor de tipo conocido.

```ts
function greet(name: string | null | undefined): string {
  if (name == null) {
    return "Hola";          // name: null | undefined
  }
  return `Hola, ${name}`;   // name: string
}
```

`x == null` es la única comparación laxa habitual: cubre `null` y `undefined` a la vez. También estrecha entre literales: tras `if (mode === "dark")`, `mode` es `"dark"` dentro del bloque.

### Truthiness (valores verdaderos y falsos)

Un `if (valor)` descarta los valores *falsy*: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined` y `NaN`.

```ts
function length(text: string | null | undefined): number {
  if (text) {
    return text.length;     // text: string
  }
  return 0;
}
```

Es cómodo, pero ojo: descarta también `""` y `0`, que pueden ser valores válidos (ver "Errores comunes").

### Type predicates (`x is T`)

Cuando la comprobación es más compleja que un `typeof`, la extraes a una función cuyo retorno es un **predicado de tipo**. Si devuelve `true`, TypeScript asume que el argumento es `T`.

```ts
type Cat = { meow(): void };
type Dog = { bark(): void };

function isCat(pet: Cat | Dog): pet is Cat {
  return "meow" in pet;
}

function speak(pet: Cat | Dog): void {
  if (isCat(pet)) {
    pet.meow();  // pet: Cat
  } else {
    pet.bark();  // pet: Dog
  }
}
```

Es la herramienta habitual para validar datos de tipo `unknown`:

```ts
function isStringArray(value: unknown): value is string[] {
  return Array.isArray(value) && value.every((item) => typeof item === "string");
}
```

### Uniones discriminadas

Es el patrón más útil. Cada miembro de la unión incluye una propiedad con un **tipo literal** distinto (el discriminante). Comprobarla estrecha toda la unión:

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function area(shape: Shape): number {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2; // shape: { kind: "circle"; ... }
  }
  return shape.side ** 2;               // shape: { kind: "square"; ... }
}
```

Con `switch` funciona igual, y es la base de las acciones de `useReducer` en React.

### Exhaustividad con `never`

`never` es el tipo sin valores posibles. Si una unión se ha estrechado por completo, lo que queda es `never`. Se aprovecha para que el compilador avise cuando aparece un miembro nuevo que nadie manejó:

```ts
function assertNever(value: never): never {
  throw new Error(`Caso no manejado: ${JSON.stringify(value)}`);
}

function perimeter(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return 2 * Math.PI * shape.radius;
    case "square":
      return 4 * shape.side;
    default:
      return assertNever(shape); // shape: never
  }
}
```

Si añades `{ kind: "triangle"; ... }` a `Shape`, `shape` en el `default` ya no es `never` y aparece un error de compilación en cada `switch` que falte actualizar.

### Assertion functions (`asserts x is T`)

Una función de aserción no devuelve un booleano: **lanza un error** si la condición falla. Si retorna con normalidad, TypeScript estrecha el tipo en las líneas siguientes.

```ts
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new TypeError("Se esperaba un string");
  }
}

function shoutValue(input: unknown): string {
  assertIsString(input);
  return input.toUpperCase(); // input: string
}
```

Se diferencia del predicado en que no se usa dentro de un `if`: el estrechamiento aplica al resto del bloque.

### `as` frente a narrowing

`as` es una **aserción de tipo**: le dices al compilador qué tipo tiene un valor, sin comprobarlo. No emite código y no cambia nada en ejecución.

```ts
const input = document.getElementById("email");

// Aserción: si no es un input, falla en ejecución al usarlo
const unsafe = input as HTMLInputElement;

// Narrowing: comprobación real
if (input instanceof HTMLInputElement) {
  console.log(input.value); // input: HTMLInputElement
}
```

El narrowing es seguro porque la comprobación existe en ejecución. La aserción traslada la responsabilidad a ti. Úsala solo cuando sabes algo que el compilador no puede saber.

-----

## Ejemplo completo

Un estado de carga modelado como unión discriminada y consumido con narrowing y comprobación exhaustiva:

```ts
type Loading = { status: "loading" };
type Success = { status: "success"; data: string[] };
type Failure = { status: "error"; error: unknown };
type LoadState = Loading | Success | Failure;

function assertNever(value: never): never {
  throw new Error(`Caso no manejado: ${JSON.stringify(value)}`);
}

function errorMessage(error: unknown): string {
  if (error instanceof Error) return error.message;
  if (typeof error === "string") return error;
  return "Error desconocido";
}

function render(state: LoadState): string {
  switch (state.status) {
    case "loading":
      return "Cargando...";
    case "success":
      return state.data.length === 0 ? "Sin resultados" : state.data.join(", ");
    case "error":
      return `Error: ${errorMessage(state.error)}`;
    default:
      return assertNever(state);
  }
}

console.log(render({ status: "success", data: ["a", "b"] })); // "a, b"
console.log(render({ status: "error", error: new Error("Fallo") })); // "Error: Fallo"
```

Puntos clave:

1. `status` es el discriminante: en cada `case`, `state` tiene la forma exacta de un miembro.
2. `error: unknown` obliga a estrechar antes de usar el valor.
3. `assertNever` hace que agregar un cuarto estado rompa la compilación hasta que se maneje.

-----

## Errores comunes

### 1. Usar truthiness y descartar valores válidos

```ts
function showCount(count: number | undefined): string {
  if (count) return `Hay ${count}`;
  return "Sin datos"; // también entra aquí cuando count es 0
}
```

**Por qué pasa:** `0`, `""` y `NaN` son falsy, igual que `undefined`.
**Solución:** compara explícitamente: `if (count !== undefined)`.

### 2. Olvidar que `typeof null` es `"object"`

```ts
function keys(value: object | null): string[] {
  if (typeof value === "object") {
    return Object.keys(value); // Error: 'object | null' no es asignable a 'object'
  }
  return [];
}
```

**Por qué pasa:** `null` también pasa la comprobación `typeof x === "object"`, y TypeScript lo refleja.
**Solución:** `if (value !== null)`, o `if (value && typeof value === "object")`.

### 3. Discriminante con tipo `string` en lugar de literal

```ts
type Bad = { kind: string; radius?: number; side?: number };
```

**Por qué pasa:** con `kind: string` no hay miembros distinguibles; TypeScript no puede estrechar por su valor.
**Solución:** un literal por miembro: `{ kind: "circle"; radius: number } | { kind: "square"; side: number }`.

### 4. Type predicate que miente

```ts
function isCat(pet: Cat | Dog): pet is Cat {
  return true; // compila, pero es falso
}
```

**Por qué pasa:** TypeScript **confía** en el predicado y no verifica el cuerpo de la función.
**Solución:** que el cuerpo compruebe de verdad lo que el tipo afirma. Ante la duda, prueba el predicado con datos reales.

### 5. Silenciar el compilador con `as`

```ts
const user = JSON.parse(raw) as User; // sin validación alguna
```

**Por qué pasa:** `as` no comprueba nada; si los datos no coinciden con `User`, el error aparecerá más adelante y lejos de la causa.
**Solución:** recibe el dato como `unknown` y valídalo con un predicado o con una librería de validación (por ejemplo Zod).

### 6. Esperar que `filter(Boolean)` estreche el tipo

```ts
const items: (string | null)[] = ["a", null, "b"];
const clean = items.filter(Boolean); // (string | null)[]
```

**Por qué pasa:** `Boolean` no es un predicado de tipo, así que el resultado conserva `null`.
**Solución:** usa un predicado explícito, `items.filter((x): x is string => x !== null)`. Desde TypeScript 5.5 también se infiere para funciones como `(x) => x !== null`.

-----

## Cuándo sí y cuándo no

**Usa narrowing para** trabajar con uniones, valores `unknown`, `null` y `undefined`, y para validar datos que vienen de fuera (APIs, `JSON.parse`, `catch`).

**Elige la herramienta según el caso:**

* Primitivos: `typeof`.
* Clases: `instanceof`.
* Objetos con formas distintas: uniones discriminadas (mejor) o `in`.
* Lógica de validación reutilizable: type predicate.
* Datos que deben ser de un tipo o el programa no puede continuar: assertion function.

**Evita `as` cuando** una comprobación en ejecución es posible. Reserva `as` para casos donde tienes información que el compilador no puede deducir.

-----

## Resumen en 5 líneas

1. Narrowing es que TypeScript reduzca un tipo amplio a uno específico según las comprobaciones del código.
2. Herramientas: `typeof`, `instanceof`, `in`, igualdad, truthiness, predicados `x is T` y funciones `asserts`.
3. `else` y las salidas tempranas (`return`, `throw`) también estrechan, con el tipo complementario.
4. Las uniones discriminadas (propiedad literal común) son el patrón más robusto para modelar variantes.
5. `never` en el `default` de un `switch` garantiza exhaustividad; `as` no verifica nada, el narrowing sí.

-----

## Para profundizar

<details>
<summary>Narrowing de `unknown`</summary>

`unknown` es el tipo seguro para valores de origen incierto: no permite ninguna operación hasta que lo estreches.

```ts
function parse(value: unknown): number {
  if (typeof value === "number") return value;
  if (typeof value === "string") return Number(value);
  throw new Error("Valor no soportado");
}
```

A diferencia de `any`, obliga a comprobar antes de usar. Es el tipo correcto para el parámetro de un `catch` (con `useUnknownInCatchVariables`, activado por `strict`) y para datos de `JSON.parse` tras validarlos.

</details>

<details>
<summary>Cómo se estrecha con `in` cuando la propiedad es opcional</summary>

Con `in`, un miembro de la unión que declara la propiedad como **opcional** puede quedar en ambas ramas:

```ts
type A = { x: number; extra?: string };
type B = { y: number };

function f(value: A | B) {
  if ("extra" in value) {
    // value: A
  } else {
    // value: A | B  (A puede existir sin 'extra')
  }
}
```

Por eso, para modelar variantes, un discriminante literal es más fiable que `in`.

</details>

<details>
<summary>Predicados inferidos (TypeScript 5.5)</summary>

Desde TypeScript 5.5, el compilador infiere un type predicate en funciones simples que devuelven el resultado de una comprobación de narrowing:

```ts
const items: (string | undefined)[] = ["a", undefined];
const clean = items.filter((x) => x !== undefined); // string[]
```

Antes de esa versión se necesitaba anotar `(x): x is string`. La inferencia solo aplica cuando la comprobación es exacta en ambos sentidos; en casos como `filter(Boolean)` o `(x) => !!x` sigue sin estrecharse.

</details>

-----

## En entrevista

### Respuesta corta (junior)

El narrowing es cuando TypeScript reduce un tipo amplio, como una unión, a uno más específico dentro de un bloque, gracias a comprobaciones como `typeof`, `instanceof` o `in`. Permite usar métodos y propiedades que solo existen en uno de los tipos sin errores del compilador. Por ejemplo, tras `typeof x === "string"`, dentro del `if` `x` es `string`.

### Respuesta ampliada (semi-senior)

* **Base:** TypeScript hace análisis de flujo de control. Cada comprobación estrecha el tipo en una rama, y `else`, `return` y `throw` estrechan el código posterior.
* **Herramientas:** `typeof` (primitivos), `instanceof` (clases), `in` (propiedades), igualdad, truthiness, type predicates (`x is T`) y assertion functions (`asserts x is T`).
* **Uniones discriminadas:** un discriminante literal común permite estrechar toda la forma del objeto. Es la base del modelado de estados y de acciones en reducers.
* **Exhaustividad:** un `default` que asigna el valor a `never` produce un error de compilación si se añade un miembro sin manejar.
* **Predicados y confianza:** el compilador no verifica el cuerpo de un predicado. Un predicado incorrecto introduce errores de tipo silenciosos.
* **`as` frente a narrowing:** `as` no genera código ni comprueba nada; el narrowing se apoya en una comprobación real en ejecución. Para datos externos se prefiere `unknown` más validación.
* **Trampas:** truthiness descarta `0` y `""`; `typeof null` es `"object"`; `filter(Boolean)` no estrecha.

### Preguntas frecuentes de seguimiento

**1. ¿Por qué no se puede usar `instanceof` con un `type` o `interface`?**
Porque los tipos se borran al compilar y no existen en ejecución. `instanceof` necesita un constructor real, es decir, una clase o una función constructora.

**2. ¿Qué diferencia hay entre un type predicate y una assertion function?**
El predicado devuelve un booleano y se usa dentro de un `if` (`x is T`). La assertion function lanza un error si falla y, si retorna, estrecha el tipo en el resto del bloque (`asserts x is T`).

**3. ¿Cómo garantizas que un `switch` cubra todos los casos de una unión?**
Asignando el valor a `never` en el `default`, por ejemplo con una función `assertNever(value: never)`. Si queda algún miembro sin manejar, el valor no es `never` y el compilador da error.

**4. ¿Por qué `if (value)` puede ser un problema con números y strings?**
Porque `0` y `""` son falsy y se descartan igual que `null` o `undefined`. Cuando esos valores son válidos, hay que comparar explícitamente (`!== undefined`, `!== null`).

**5. ¿Qué es un discriminante y por qué debe ser un literal?**
Es la propiedad común (por ejemplo `kind` o `status`) cuyo valor identifica cada miembro de la unión. Debe tener tipo literal para que el compilador pueda asociar cada valor con un solo miembro. Con `string` no puede.

**6. ¿Por qué `as` es más peligroso que un type guard?**
Porque no verifica nada: si el valor real no coincide con el tipo afirmado, el error ocurre más tarde y lejos del origen. Un type guard realiza la comprobación en ejecución y el tipo queda respaldado por ella.

-----

## Siguiente lección

Ya sabes estrechar tipos para trabajar con uniones y datos inciertos. A continuación, veremos cómo describir objetos con más detalle (propiedades opcionales, de solo lectura, index signatures y más): [Advanced Object Types](08-Advanced%20Object%20Types.md).
