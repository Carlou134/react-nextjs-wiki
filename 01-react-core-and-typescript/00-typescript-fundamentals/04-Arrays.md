# Arrays y tuplas: tipar listas

## En una frase

Un array tipado (`T[]`) es una lista de longitud variable donde **todos los elementos son de un mismo tipo**; una tupla (`[A, B]`) es una lista de longitud y posiciones fijas donde **cada posición tiene su propio tipo**.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué son los tipos primitivos y las anotaciones de tipo: [Types](01-Types.md).
* Cómo se tipan parámetros y retornos de funciones: [Functions](03-Functions.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Anotación de tipo:** declaración explícita del tipo de una variable, como `let names: string[]`.
* **Inferencia:** deducción automática del tipo que hace TypeScript a partir del valor inicial.
* **Tupla:** array cuyo tipo fija la cantidad de elementos y el tipo de cada posición.
* **Readonly:** modificador que impide mutar un valor a través de ese tipo. Solo existe en tiempo de compilación.
* **Type guard (guarda de tipo):** función o expresión que le permite a TypeScript acotar un tipo dentro de un bloque.

-----

## El problema

En JavaScript un array acepta cualquier mezcla de valores, y los errores aparecen en ejecución:

```js
const names = ['Danny', 'Samantha'];
names.push(666);                       // JavaScript lo acepta
names.map((n) => n.toUpperCase());     // TypeError: n.toUpperCase is not a function
```

Mantener la consistencia de una lista implica controlar el tipo de **cada** elemento que entra y sale. TypeScript lo hace en compilación: si declaras qué tipo contiene la lista, rechaza cualquier elemento que no encaje.

-----

## Cómo funciona

### Dos sintaxis equivalentes: `T[]` y `Array<T>`

```ts
let names: string[] = ['Danny', 'Samantha'];
let sameNames: Array<string> = ['Danny', 'Samantha'];
```

Significan exactamente lo mismo. `T[]` es la forma habitual; `Array<T>` es la forma genérica y aparece en APIs y mensajes de error. Elige una y mantenla en todo el proyecto.

La diferencia práctica es de legibilidad cuando el tipo del elemento es compuesto:

```ts
let a: (string | number)[] = [];      // los paréntesis son obligatorios
let b: Array<string | number> = [];   // sin paréntesis, más claro
```

Sin paréntesis, `string | number[]` significa "un `string` o un array de `number`".

TypeScript valida tanto la asignación como las mutaciones posteriores:

```ts
let names: string[] = [1, 2, 3];   // Error: number no es asignable a string

let ok: string[] = ['Damien'];
ok.push(666);                      // Error: 666 no es string
```

### Arrays multidimensionales

Un array de arrays se escribe encadenando corchetes. `string[][]` es `(string[])[]`: cada elemento es un `string[]`.

```ts
let grid: string[][] = [['a', 'b'], ['c', 'd']];
let cell: string = grid[1][0];   // 'c'
```

### Inferencia: siempre `T[]`, nunca tupla

```ts
let examAnswers = [true, false, false];   // boolean[]
examAnswers[3] = true;                    // válido: un array puede crecer
```

TypeScript infiere el tipo **menos restrictivo**: `boolean[]`, no `[boolean, boolean, boolean]`. Si quieres una tupla, debes anotarla o usar `as const` (más abajo).

Los métodos que combinan arrays también devuelven arrays, no tuplas:

```ts
let tup: [number, number, number] = [1, 2, 3];
let joined = tup.concat([4, 5, 6]);   // number[]
```

### Arrays vacíos y `never[]`

Un `[]` es asignable a cualquier tipo de array:

```ts
let names: string[] = [];
let numbers: number[] = [];
names.push('Isabella');
numbers.push(30);
```

El problema aparece cuando no hay anotación y TypeScript no tiene contexto. Con `strictNullChecks` activado (incluido en `strict`), un `[]` sin contexto puede inferirse como `never[]`: una lista que no admite ningún elemento, porque `never` es el tipo sin valores.

```ts
function identity<T>(value: T): T {
  return value;
}

const list = identity([]);   // never[]
list.push('x');              // Error: string no es asignable a never
```

Este es el motivo por el que en React se escribe `useState<string[]>([])`. La solución es anotar el tipo:

```ts
const list: string[] = identity([]);   // ahora T se infiere como string[]
```

Una variable declarada con `let items = []` se comporta distinto: bajo `noImplicitAny`, TypeScript usa un tipo "evolutivo" que se va ajustando con cada `push`. Es válido, pero frágil; anotar (`let items: string[] = []`) es más claro.

### Tuplas

Una tupla fija longitud, orden y tipo por posición:

```ts
let ourTuple: [string, number, boolean] = ['Is', 7, false];

let numbersTuple: [number, number, number] = [1, 2, 3, 4];     // Error: la tupla tiene 3 elementos
let mixedTuple: [number, string, boolean] = ['hi', 3, true];   // Error: los tipos no coinciden por posición
```

En ejecución una tupla es un array común (tiene `.length`, se accede por índice). La diferencia existe solo en el sistema de tipos: un `string[]` **no** es asignable a `[string, string]`, porque TypeScript no puede garantizar la longitud.

```ts
let tup: [string, string] = ['hi', 'bye'];
let arr: string[] = ['there', 'there'];
tup = ['there', 'there'];   // válido
tup = arr;                  // Error: string[] no es asignable a [string, string]
```

Accede fuera de rango y TypeScript avisa:

```ts
const pair: [string, number] = ['a', 1];
pair[2];   // Error: la tupla tiene longitud 2 y no hay elemento en el índice 2
```

Las tuplas admiten tres extensiones útiles:

```ts
type Named = [name: string, age: number];           // elementos con etiqueta (solo documentan)
type MaybeAge = [name: string, age?: number];       // elemento opcional
type Scores = [label: string, ...values: number[]]; // resto: una etiqueta y luego cero o más números
```

Un caso conocido: en una tupla **mutable**, `push` sí compila, aunque cambie la longitud real.

```ts
const point: [number, number] = [1, 2];
point.push(3);   // compila; el tipo sigue siendo [number, number]
```

Para evitarlo, usa `readonly [number, number]`.

### Arrays de solo lectura: `readonly`

```ts
function sum(values: readonly number[]): number {
  // values.push(1);   // Error: 'push' no existe en readonly number[]
  return values.reduce((acc, n) => acc + n, 0);
}

const nums: number[] = [1, 2, 3];
sum(nums);   // válido: un array mutable es asignable a uno de solo lectura
```

`readonly number[]` (equivalente a `ReadonlyArray<number>`) elimina los métodos que mutan (`push`, `pop`, `splice`, `sort`, `reverse`...). Conserva los que devuelven un array nuevo (`map`, `filter`, `slice`, `concat`).

La asignación solo funciona en un sentido: un `number[]` se puede pasar donde se espera `readonly number[]`, pero no al revés. Declarar un parámetro como `readonly` es una promesa a quien llama de que la función no mutará su lista. Recuerda que es una protección de compilación: no congela el array en ejecución (para eso existe `Object.freeze`).

### Métodos de orden superior tipados

Los tipos de `map`, `filter` y `reduce` se infieren a partir del array y de la función que pasas.

```ts
const nums = [1, 2, 3];

const doubled = nums.map((n) => n * 2);           // number[]
const labels = nums.map((n) => `#${n}`);          // string[]
const even = nums.filter((n) => n % 2 === 0);     // number[]
const total = nums.reduce((acc, n) => acc + n, 0); // number
```

`map` devuelve un array del tipo que retorne el callback. Si quieres tuplas, indícalo en el retorno:

```ts
const pairs = ['a', 'bb'].map((s): [string, number] => [s, s.length]);
// [string, number][]
```

`reduce` infiere el acumulador a partir del valor inicial. Cuando el valor inicial es demasiado estrecho (por ejemplo `{}`), pasa el genérico:

```ts
type User = { id: number; role: 'admin' | 'user' };

const users: User[] = [
  { id: 1, role: 'admin' },
  { id: 2, role: 'user' },
];

const byRole = users.reduce<Record<string, User[]>>((acc, user) => {
  (acc[user.role] ??= []).push(user);
  return acc;
}, {});
```

`filter` **no** cambia el tipo por sí solo. Para acotarlo, el callback debe ser una guarda de tipo:

```ts
const mixed: (string | null)[] = ['a', null, 'b'];

const strings = mixed.filter((x): x is string => x !== null);   // string[]
const stillMixed = mixed.filter(Boolean);                       // (string | null)[]
```

Desde TypeScript 5.5, en casos simples como `x => x !== null` el compilador infiere el predicado automáticamente. `filter(Boolean)` sigue sin acotar el tipo.

### Desestructuración

Con arrays, la desestructuración asigna por posición. Con tuplas, cada variable recibe el tipo de su posición:

```ts
const tuple: [string, number] = ['Ana', 30];
const [name, age] = tuple;   // name: string, age: number

const [head, ...tail] = [1, 2, 3];   // head: number, tail: number[]

const [x, y, z] = tuple;   // Error: la tupla no tiene elemento en el índice 2
```

Con un `string[]` (no tupla), TypeScript da por hecho que el elemento existe:

```ts
const items: string[] = [];
const [first] = items;   // first: string, aunque en ejecución sea undefined
```

Esa confianza se puede quitar activando `noUncheckedIndexedAccess` en el [tsconfig](02-Archivo%20tsconfig.md): entonces `first` es `string | undefined`, y `items[0]` también.

### Arrays de uniones

Hay dos formas distintas, y no significan lo mismo:

```ts
let mixed: (string | number)[] = ['a', 1, 'b'];   // cada elemento es string o number

function addItem(either: string[] | number[]) {   // o todo strings, o todo numbers
  either.push('x');   // Error: 'x' no es asignable a never
}
```

* `(string | number)[]` admite elementos mezclados. Al leer un elemento, debes acotarlo (`typeof`) antes de usar métodos propios de `string` o `number`.
* `string[] | number[]` es una lista homogénea, pero de tipo desconocido a priori. No puedes hacer `either.push('x')`, porque TypeScript no sabe cuál de las dos es. Ojo: si asignas un valor al declarar la variable (`let either: string[] | number[] = ['a']`), TypeScript reduce el tipo a `string[]` por la asignación y `push('x')` sí compila; el error aparece cuando el valor llega desde fuera, como un parámetro.

### `as const`: literales y tuplas de solo lectura

Por defecto, un array literal se ensancha a `T[]`. Con `as const`, TypeScript conserva los valores exactos y lo trata como tupla de solo lectura:

```ts
const roles = ['admin', 'user', 'guest'] as const;
// readonly ['admin', 'user', 'guest']

type Role = (typeof roles)[number];   // 'admin' | 'user' | 'guest'
```

`typeof roles` es el tipo del valor, e indexarlo con `[number]` extrae el tipo de sus elementos. Es la forma habitual de tener **una sola fuente de verdad** para una lista de valores y su tipo de unión.

También resuelve el problema de las tuplas inferidas como arrays:

```ts
const coords = [40, 43.2] as const;   // readonly [40, 43.2]
```

### Parámetros rest y spread con tuplas

Un parámetro rest recibe los argumentos restantes como array, y se anota con la misma sintaxis:

```ts
function smush(first: string, ...others: string[]): string {
  return others.reduce((acc, s) => acc.concat(s), first);
}

smush('a', 'h', 'h', '!');   // 'ahh!'
smush(1, 2, 3);              // Error: number no es asignable a string
```

Una tupla permite usar spread al llamar una función con muchos parámetros, verificando cada posición:

```ts
function route(fromLat: number, fromLon: number, toLat: number, toLon: number) {
  /* ... */
}

const start: [number, number] = [40, -73];
const end: [number, number] = [25, -71];

route(...start, ...end);   // válido: 2 + 2 argumentos, todos number
```

Con `start: number[]` esta llamada fallaría, porque TypeScript no puede saber que el array aporta exactamente dos argumentos.

-----

## Ejemplo completo

Un catálogo con una lista de valores permitidos, una tupla de retorno y un resumen por categoría:

```ts
const categories = ['book', 'game', 'music'] as const;
type Category = (typeof categories)[number];

type Product = { id: number; name: string; price: number; category: Category };

const catalog: readonly Product[] = [
  { id: 1, name: 'TypeScript Handbook', price: 30, category: 'book' },
  { id: 2, name: 'Chess', price: 15, category: 'game' },
  { id: 3, name: 'Kind of Blue', price: 12, category: 'music' },
  { id: 4, name: 'Clean Code', price: 40, category: 'book' },
];

// Tupla como retorno: [mínimo, máximo]
function priceRange(products: readonly Product[]): [number, number] {
  const prices = products.map((p) => p.price);
  return [Math.min(...prices), Math.max(...prices)];
}

const [min, max] = priceRange(catalog);   // min: number, max: number

// reduce con genérico explícito: total por categoría
const totalByCategory = catalog.reduce<Record<Category, number>>(
  (acc, product) => {
    acc[product.category] += product.price;
    return acc;
  },
  { book: 0, game: 0, music: 0 },
);

// filter + map: nombres de productos caros (string[])
const expensiveNames = catalog
  .filter((p) => p.price > 20)
  .map((p) => p.name);

console.log(min, max, totalByCategory, expensiveNames);
// 12 40 { book: 70, game: 15, music: 12 } [ 'TypeScript Handbook', 'Clean Code' ]
```

Puntos clave:

1. `categories` es la única fuente de verdad: el tipo `Category` se deriva de la lista.
2. `readonly Product[]` declara que ni la función ni el módulo mutarán el catálogo.
3. `priceRange` devuelve una tupla, así que la desestructuración da `number` en ambas posiciones.
4. `reduce<Record<Category, number>>` obliga a que el valor inicial tenga todas las categorías.

-----

## Errores comunes

### 1. `[]` sin anotación: `never[]` o `any[]`

```ts
const list = identity([]);   // never[]
list.push('x');              // Error
```

**Por qué pasa:** sin contexto, TypeScript no tiene evidencia del tipo de los elementos y, con `strictNullChecks`, elige `never[]` en algunos contextos (por ejemplo, genéricos como `useState([])`).
**Solución:** anota el tipo: `const list: string[] = []` o `useState<string[]>([])`.

### 2. Esperar una tupla y obtener un array

```ts
const point = [10, 20];              // number[]
const [x, y]: [number, number] = point;   // Error: number[] no es asignable a [number, number]
```

**Por qué pasa:** la inferencia siempre devuelve `T[]`.
**Solución:** anota la variable (`const point: [number, number] = [10, 20]`) o usa `as const`.

### 3. Confundir `(A | B)[]` con `A[] | B[]`

```ts
function addItem(either: string[] | number[]) {
  either.push('b');   // Error
}
```

**Por qué pasa:** el tipo es "una lista de strings **o** una de números"; `push` tendría que aceptar un valor que sirva para ambas, es decir `string & number`, que es `never`.
**Solución:** si la lista puede mezclar tipos, usa `(string | number)[]`.

### 4. Suponer que `filter` acota el tipo

```ts
const items: (string | undefined)[] = ['a', undefined];
const defined = items.filter((x) => x);   // (string | undefined)[]: la comprobación de truthiness no acota
```

**Por qué pasa:** `filter` solo cambia el tipo si el callback es una guarda de tipo.
**Solución:** escribe el predicado: `items.filter((x): x is string => x !== undefined)`.

### 5. Confiar en `arr[i]` sin comprobar

```ts
const items: string[] = [];
items[0].toUpperCase();   // compila; falla en ejecución
```

**Por qué pasa:** por defecto TypeScript asume que todo índice existe.
**Solución:** activa `noUncheckedIndexedAccess` y comprueba (`items[0]?.toUpperCase()`).

### 6. Creer que `readonly` protege en ejecución

```ts
const nums: readonly number[] = [1, 2];
(nums as number[]).push(3);   // compila y muta
```

**Por qué pasa:** `readonly` es solo información de tipos; desaparece al compilar.
**Solución:** úsalo como contrato de diseño. Si necesitas inmutabilidad real, crea copias o usa `Object.freeze`.

-----

## Cuándo sí y cuándo no

**Usa `T[]` cuando** la lista tiene longitud variable y todos los elementos cumplen el mismo rol: usuarios, precios, tareas.

**Usa una tupla cuando** la longitud es fija y cada posición significa algo distinto: `[valor, setter]`, `[lat, lon]`, un par clave-valor.

**Usa `readonly` en parámetros** que la función solo lee, y en datos constantes compartidos.

**Usa `as const`** para listas de valores fijos de las que quieres derivar un tipo de unión.

**No uses una tupla larga** (más de tres o cuatro posiciones): un objeto con propiedades con nombre es más legible (`{ lat, lon }` en lugar de `[lat, lon, ...]`).

**No dependas de que una tupla mutable garantice su longitud**: `push` la rompe sin error de compilación.

-----

## Resumen en 5 líneas

1. `T[]` y `Array<T>` son equivalentes; usa paréntesis en uniones: `(string | number)[]`.
2. La inferencia siempre produce `T[]`; para una tupla, anótala o usa `as const`.
3. Un `[]` sin contexto puede inferirse como `never[]`: anota el tipo (`string[]`).
4. `readonly T[]` impide mutar por tipo; `T[]` es asignable a él, pero no al revés.
5. `filter` solo acota con una guarda de tipo; `reduce` infiere el acumulador del valor inicial o del genérico que indiques.

-----

## Para profundizar

<details>
<summary>Diferencia entre `T[]` y una tupla en asignabilidad</summary>

Una tupla es un subtipo de `T[]` (un `[string, string]` es asignable a `string[]`), pero no al revés. Por eso puedes pasar una tupla a una función que espera `readonly string[]`, y no puedes asignar un `string[]` a una tupla.

</details>

<details>
<summary>`Array.isArray` y `unknown`</summary>

Cuando un valor llega como `unknown` (por ejemplo, de `JSON.parse`), `Array.isArray(value)` lo acota a `any[]`, no a un tipo concreto. Sus elementos siguen sin verificarse: hace falta comprobar cada elemento (`typeof`) o validar con una librería antes de tratarlos como `string[]`.

```ts
function toStrings(value: unknown): string[] {
  if (!Array.isArray(value)) return [];
  return value.filter((x): x is string => typeof x === 'string');
}
```

</details>

<details>
<summary>`noUncheckedIndexedAccess`</summary>

Con esta opción del `tsconfig`, leer por índice (`arr[i]`) devuelve `T | undefined`, lo que obliga a comprobar la existencia. No afecta a las tuplas en posiciones válidas (`pair[0]` sigue siendo `string`). Aumenta la seguridad a costa de más comprobaciones, y suele valer la pena en proyectos nuevos.

</details>

<details>
<summary>Tuplas en React: `useState`</summary>

`useState` devuelve una tupla `[T, Dispatch<SetStateAction<T>>]`. Por eso la desestructuración `const [count, setCount] = useState(0)` conserva un tipo distinto para cada posición. Si un Hook propio devuelve `[value, setValue]` sin anotar el retorno, TypeScript infiere un array de la unión de ambos tipos (por ejemplo `(T | Dispatch<SetStateAction<T>>)[]`) y se pierde esa precisión: hay que anotar el retorno como tupla o usar `as const`.

</details>

-----

## En entrevista

### Respuesta corta (junior)

Un array se tipa con `T[]` o `Array<T>` y todos sus elementos deben ser de tipo `T`; TypeScript rechaza asignar o agregar otros tipos. Una tupla, como `[string, number]`, fija la longitud y el tipo de cada posición. La inferencia siempre da un array, así que las tuplas se anotan explícitamente.

### Respuesta ampliada (semi-senior)

* **Sintaxis:** `T[]` y `Array<T>` son equivalentes. En uniones se usan paréntesis: `(A | B)[]` es distinto de `A[] | B[]`.
* **Inferencia:** un literal de array se infiere como `T[]`; una tupla exige anotación o `as const`, que además la hace `readonly` y con tipos literales.
* **Vacíos:** `[]` sin contexto puede dar `never[]`, por lo que en genéricos como `useState<string[]>([])` se anota el tipo.
* **Tuplas:** longitud y orden fijos, con etiquetas, elementos opcionales y resto. `T[]` no es asignable a una tupla; una tupla sí lo es a `T[]`. Una tupla mutable no impide `push` en tipos.
* **`readonly`:** un array mutable es asignable a uno `readonly`, no al revés. Es una restricción solo de compilación.
* **Métodos:** `map` infiere el tipo del retorno; `reduce` infiere el acumulador del valor inicial (o del genérico); `filter` solo acota con una guarda de tipo (el compilador la infiere en casos simples desde TypeScript 5.5).
* **Índices:** por defecto `arr[i]` es `T`; con `noUncheckedIndexedAccess` es `T | undefined`.
* **`as const` + `typeof x[number]`:** derivar un tipo de unión desde una lista de valores mantiene una única fuente de verdad.

### Preguntas frecuentes de seguimiento

**1. ¿Cuál es la diferencia entre `string[]` y `Array<string>`?**
Ninguna en tipos: son el mismo. `string[]` es azúcar sintáctica de `Array<string>`. La elección es de estilo, y conviene ser consistente en el proyecto.

**2. ¿Cuál es la diferencia entre `(string | number)[]` y `string[] | number[]`?**
La primera es una lista que puede mezclar strings y números. La segunda es una lista donde todo es string o todo es number. En esta última no se puede hacer `push` sin acotar primero.

**3. ¿Cuándo usarías una tupla en lugar de un objeto?**
Cuando son pocos valores y la posición es suficientemente obvia, como `[valor, setter]` o `[lat, lon]`. Si hay más de tres o cuatro campos, o las posiciones no son evidentes, un objeto con nombres es más legible.

**4. ¿Por qué `useState([])` da problemas de tipos?**
Sin contexto, el `[]` se infiere como `never[]` y no admite ningún elemento. Se soluciona con `useState<string[]>([])` (o el tipo que corresponda).

**5. ¿Qué hace `as const` en un array?**
Lo convierte en una tupla `readonly` con tipos literales: `['a', 'b'] as const` es `readonly ['a', 'b']`. Permite derivar una unión con `(typeof arr)[number]`.

**6. ¿Cómo haces que `filter` quite los `null` del tipo?**
Con una guarda de tipo: `.filter((x): x is string => x !== null)`. Desde TypeScript 5.5 el compilador infiere el predicado en casos simples; `filter(Boolean)` no acota el tipo.

-----

## Siguiente lección

Ya sabes tipar listas y tuplas. El siguiente paso es dar nombre a las formas de datos que se repiten con alias, interfaces y otros tipos personalizados: [Custom Types](05-Custom%20Types.md).
