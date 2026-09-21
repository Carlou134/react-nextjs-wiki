# Tipos personalizados: alias, interfaces y objetos

## En una frase

Un **tipo personalizado** es un tipo que defines tú combinando los predefinidos; las herramientas principales son los tipos de objeto, `type` (alias), `interface`, los modificadores `?` y `readonly`, las intersecciones y las firmas de índice.

-----

## Antes de empezar

Conviene que ya sepas:

* Los tipos primitivos y las anotaciones de tipo: [Types](01-Types.md).
* Cómo se tipan las funciones: [Functions](03-Functions.md).
* Cómo se tipan los arrays y las tuplas: [Arrays](04-Arrays.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Tipo de objeto:** tipo que describe las propiedades de un objeto y el tipo de cada una.
* **Alias de tipo:** nombre que se le da a cualquier tipo con `type Nombre = ...`.
* **Interface:** declaración que nombra la forma de un objeto con `interface Nombre { ... }`.
* **Tipado estructural:** TypeScript compara tipos por su **forma** (las propiedades que tienen), no por el nombre con que fueron declarados.

-----

## El problema

Los tipos predefinidos no bastan para describir datos reales. Una anotación en línea funciona una vez, pero se repite y se desincroniza:

```typescript
function greet(user: { name: string; age: number }) { /* ... */ }
function save(user: { name: string; age: number }) { /* ... */ }
function render(user: { name: string; age: number; email: string }) { /* ... */ } // ¿otro tipo o un descuido?
```

Se necesita **nombrar** una forma una sola vez y reutilizarla, y poder expresar detalles como propiedades opcionales, de solo lectura o en número variable.

-----

## Cómo funciona

### Tipos de objeto

Una anotación de objeto se parece a un literal, pero en lugar de valores lleva tipos:

```typescript
let aPerson: { name: string; age: number };

aPerson = { name: 'Ana', age: 22 };                 // válido
aPerson = { name: 'Kushim', yearsOld: 5000 };       // error: falta "age" y "yearsOld" no existe
aPerson = { name: 'Ana', age: 'veintidós' };        // error: "age" debe ser number
```

Las propiedades pueden ser de cualquier tipo: primitivos, arrays, funciones u otros objetos.

Un detalle importante: al asignar un **literal de objeto** directamente, TypeScript rechaza propiedades sobrantes (*excess property check*). Si el valor viene de una variable, esa revisión no se aplica y basta con que tenga al menos las propiedades requeridas (tipado estructural):

```typescript
type Person = { name: string; age: number };

const withExtra = { name: 'Ana', age: 22, city: 'Lima' };
const p: Person = withExtra;                              // válido: la variable tiene name y age
const q: Person = { name: 'Ana', age: 22, city: 'Lima' }; // error: literal con propiedad sobrante
```

### Alias de tipo (`type`)

Un **alias** da un nombre a cualquier tipo:

```typescript
type Person = { name: string; age: number };

type Company = {
  companyName: string;
  boss: Person;
  employees: Person[];
  employeeOfTheMonth: Person;
  moneyEarned: number;
};
```

Un alias **no crea un tipo nuevo**, solo un nombre. Por eso dos alias del mismo tipo son intercambiables:

```typescript
type MyString = string;
type MyOtherString = string;

const a: MyString = 'test';
const b: MyOtherString = a; // válido: ambos son string
```

A diferencia de `interface`, un alias sirve para **cualquier** tipo: primitivos, uniones, tuplas y funciones.

```typescript
type Id = string | number;                       // unión (siguiente lección)
type Coordinates = [number, number];            // tupla
type StringsToNumber = (a: string, b: string) => number; // función
```

### Interfaces

Una `interface` describe la forma de un objeto:

```typescript
interface User {
  name: string;
  age: number;
}

const user: User = { name: 'Ana', age: 22 };
```

Se pueden **extender** con `extends`, y una interface puede extender varias:

```typescript
interface Admin extends User {
  permissions: string[];
}

const admin: Admin = { name: 'Eva', age: 40, permissions: ['delete'] };
```

Una interface también puede extender un alias de tipo de objeto, y un alias puede combinar interfaces con intersección (ver más abajo).

### Propiedades opcionales (`?`)

Con `?` la propiedad puede omitirse. Su tipo pasa a ser `T | undefined`:

```typescript
interface Profile {
  name: string;
  bio?: string; // string | undefined
}

const p1: Profile = { name: 'Ana' };                    // válido
const p2: Profile = { name: 'Ana', bio: 'Desarrolladora' }; // válido

p1.bio.toUpperCase(); // error: "bio" puede ser undefined
p1.bio?.toUpperCase(); // válido
```

Con la opción `exactOptionalPropertyTypes` (desactivada por defecto), `bio?: string` prohíbe asignar `undefined` de forma explícita. Es un detalle que solo importa si activas esa opción.

### Propiedades de solo lectura (`readonly`)

`readonly` impide reasignar una propiedad después de crear el objeto:

```typescript
interface Point {
  readonly x: number;
  readonly y: number;
}

const origin: Point = { x: 0, y: 0 };
origin.x = 5; // error: "x" es de solo lectura
```

Dos límites que conviene conocer:

* Es una comprobación **solo en compilación**. En ejecución la propiedad sigue siendo modificable.
* Es **superficial**: impide reasignar la propiedad, pero no protege el contenido de lo que apunta. Un `readonly items: string[]` no permite `obj.items = []`, aunque sí `obj.items.push('x')`. Para proteger el array, usa `readonly string[]`.

### Intersecciones (`&`)

Una **intersección** combina varios tipos en uno que debe cumplir **todos**:

```typescript
type Named = { name: string };
type Aged = { age: number };

type Person = Named & Aged; // { name: string; age: number }

const p: Person = { name: 'Ana', age: 22 }; // válido
```

Si dos tipos declaran la misma propiedad con tipos incompatibles, la propiedad resultante es `never` y no se puede asignar ningún valor:

```typescript
type A = { id: string };
type B = { id: number };
type C = A & B; // id: string & number = never
```

### Firmas de índice (index signatures)

Cuando no conoces los nombres de las propiedades, pero sí el tipo de las claves y de los valores, usa una **firma de índice**:

```typescript
interface Scores {
  [student: string]: number; // "student" es solo un nombre descriptivo
}

const scores: Scores = { ana: 9, luis: 7 };
scores.eva = 10;          // válido
scores.pedro = 'alto';    // error: el valor debe ser number
```

`Record<string, number>` es una forma equivalente y más corta. Ten en cuenta que, con una firma de índice, TypeScript asume que **cualquier** clave existe: `scores.inexistente` tiene tipo `number` aunque en ejecución sea `undefined`. La opción `noUncheckedIndexedAccess` (en `tsconfig`) corrige eso y lo tipa como `number | undefined`.

Si algunas propiedades son conocidas, deben ser compatibles con el tipo de la firma:

```typescript
interface Config {
  [key: string]: string | number;
  version: number; // válido: number es parte de string | number
}
```

### Tipos de función

Un tipo de función indica los tipos de los parámetros y del retorno. Se escribe con sintaxis parecida a la de una flecha, y se usa mucho para callbacks:

```typescript
type Comparator = (a: string, b: string) => number;

const byLength: Comparator = (x, y) => x.length - y.length;
```

Los nombres de los parámetros del tipo son solo documentación: no tienen que coincidir con los de la función asignada. Cada parámetro debe llevar nombre **y** tipo. Escribir `(string) => number` es un error de concepto: TypeScript lo interpreta como un parámetro **llamado** `string` de tipo implícito `any`, lo que falla con `noImplicitAny`. Lo correcto es `(text: string) => number`.

### Enums

Un **enum** enumera los valores posibles de una variable:

```typescript
enum Direction {
  North = 'NORTH',
  South = 'SOUTH',
  East = 'EAST',
  West = 'WEST',
}

let heading: Direction = Direction.North; // válido
heading = 'SOUTH';                        // error: hay que usar Direction.South
```

Hay dos tipos:

* **Numéricos:** `enum Direction { North, South, East, West }` asigna 0, 1, 2 y 3 automáticamente; puedes fijar el inicio (`North = 7`) o cada valor.
* **De cadenas:** cada miembro lleva su valor explícito, como arriba.

Se recomiendan los de cadenas: al depurar, `'NORTH'` dice más que `0`, y son más estrictos. Los numéricos permiten conversiones implícitas con `number` (por ejemplo, un `number` cualquiera es asignable a un enum numérico). Desde TypeScript 5.0, asignar un literal numérico fuera del rango del enum sí da error, pero un valor de tipo `number` genérico sigue pasando.

Ten presente que un enum **genera código JavaScript** en ejecución (un objeto), a diferencia de `type` e `interface`, que desaparecen al compilar. Muchos equipos prefieren una unión de literales (`type Direction = 'NORTH' | 'SOUTH'`, ver [Union Types](06-Union%20Types.md)) o un objeto `as const`.

### Genéricos

Un **genérico** es un tipo o función con **parámetros de tipo**: un marcador (por convención `T`) que se sustituye al usarlo. Ya conoces uno: `Array<T>`.

```typescript
type Family<T> = {
  parents: [T, T];
  mate: T;
  children: T[];
};

const stringFamily: Family<string> = {
  parents: ['a', 'b'],
  mate: 'c',
  children: ['d', 'e'],
};
```

`Family<T>` no se puede usar sin indicar `T`. Con las funciones, el genérico conecta el tipo de la entrada con el de la salida:

```typescript
function getFilledArray<T>(value: T, n: number): T[] {
  return Array(n).fill(value);
}

const cheeses = getFilledArray('cheese', 3); // string[]; T se infiere
const ones = getFilledArray<number>(1, 3);   // T indicado de forma explícita
```

Los genéricos tienen su propia lección más adelante; aquí basta con entender el concepto de marcador de tipo.

-----

## Ejemplo completo

Un modelo de tienda que combina varias piezas:

```typescript
type Currency = 'USD' | 'EUR';

interface Entity {
  readonly id: number;
}

interface Product extends Entity {
  name: string;
  price: number;
  description?: string;
}

type Timestamps = { createdAt: Date; updatedAt: Date };

type StoredProduct = Product & Timestamps;

interface PriceList {
  [productName: string]: { amount: number; currency: Currency };
}

type Formatter = (product: Product) => string;

const format: Formatter = (p) => `${p.name}: ${p.price}`;

const product: StoredProduct = {
  id: 1,
  name: 'Teclado',
  price: 49.9,
  createdAt: new Date(),
  updatedAt: new Date(),
};

const prices: PriceList = {
  Teclado: { amount: 49.9, currency: 'USD' },
};

console.log(format(product));          // válido: StoredProduct es asignable a Product
// product.id = 2;                     // error: "id" es de solo lectura
// product.description.length;         // error: puede ser undefined
```

Puntos clave:

1. `interface` con `extends` modela una jerarquía; `type` con `&` compone piezas independientes.
2. `id` es `readonly` porque no debe cambiar tras crearse.
3. `description?` puede faltar, y TypeScript obliga a comprobarlo antes de usarlo.
4. `format` acepta un `StoredProduct` porque tiene todo lo que exige `Product` (tipado estructural).

-----

## Errores comunes

### 1. Esperar que `type` o `interface` existan en ejecución

```typescript
interface User { name: string }
if (value instanceof User) {} // error: "User" solo se refiere a un tipo
```

**Por qué pasa:** los tipos se borran al compilar; no hay ningún objeto `User` en JavaScript.
**Solución:** valida en ejecución con `typeof`, `in` o una función guardia (ver [Type Narrowing](07-Type%20Narrowing.md)).

### 2. Confiar en que una API externa cumple el tipo

```typescript
const user = (await response.json()) as User; // TypeScript lo acepta sin comprobar nada
```

**Por qué pasa:** `as` es una promesa tuya al compilador, no una validación.
**Solución:** valida los datos externos en el borde de la aplicación (por ejemplo, con una librería de esquemas) antes de tratarlos como `User`.

### 3. Olvidar comprobar una propiedad opcional

```typescript
function initials(p: { name: string; bio?: string }) {
  return p.bio.slice(0, 1); // error con strict: "bio" puede ser undefined
}
```

**Por qué pasa:** `bio?: string` significa `string | undefined`.
**Solución:** `p.bio?.slice(0, 1)` o una comprobación previa.

### 4. Creer que `readonly` protege en ejecución

```typescript
const point: Readonly<{ x: number }> = { x: 1 };
(point as { x: number }).x = 9; // compila; en ejecución cambia
```

**Por qué pasa:** `readonly` solo lo comprueba el compilador. **Solución:** para inmutabilidad real usa `Object.freeze` (también superficial) o estructuras inmutables.

### 5. Intersectar propiedades incompatibles

```typescript
type Broken = { id: string } & { id: number }; // id: never
```

**Por qué pasa:** el valor debería ser `string` y `number` a la vez. **Solución:** revisa el diseño; si las formas son alternativas, usa una unión.

-----

## Cuándo sí y cuándo no

**`interface` cuando:**

* Describes la forma de un objeto o una jerarquía con `extends`.
* Quieres que otros puedan ampliarla (declaration merging), típico en librerías.
* Una clase la va a implementar (`implements`).

**`type` cuando:**

* Necesitas uniones, tuplas, primitivos con nombre, tipos de función o tipos derivados (mapped y condicionales).
* Quieres componer con intersecciones.

**Regla práctica:** en código de aplicación ambos funcionan para objetos. Elige uno y mantén la coherencia del equipo. Una convención habitual es `interface` para formas de objeto públicas y `type` para todo lo demás.

**Enums:** úsalos con moderación. Una unión de literales suele ser más simple y no genera código.

-----

## Resumen en 5 líneas

1. Un tipo de objeto describe propiedades y sus tipos; TypeScript compara por forma (tipado estructural).
2. `type` da nombre a cualquier tipo; `interface` nombra la forma de un objeto y se extiende con `extends`.
3. `?` hace opcional una propiedad (`T | undefined`); `readonly` impide reasignarla, solo en compilación.
4. `A & B` exige cumplir ambos tipos; `[key: string]: T` describe objetos con claves dinámicas.
5. Los tipos se borran al compilar: no validan datos en ejecución.

-----

## Para profundizar

<details>
<summary>Declaration merging: cómo se fusionan las interfaces</summary>

Si declaras dos `interface` con el mismo nombre en el mismo ámbito, TypeScript las **fusiona** en una sola:

```typescript
interface Window {
  appVersion: string;
}
interface Window {
  userId: number;
}
// Window ahora tiene ambas propiedades
```

Con `type`, declarar el mismo nombre dos veces da error (`Duplicate identifier`). La fusión permite ampliar tipos de librerías o globales (por ejemplo, agregar propiedades a `Window`), pero también puede causar fusiones accidentales si repites un nombre por descuido.

</details>

<details>
<summary>`extends` frente a intersección `&`</summary>

Ambos combinan formas, pero difieren al haber conflictos:

```typescript
interface A { id: string }
interface B extends A { id: number } // error en la declaración: incompatible con A

type C = { id: string } & { id: number }; // sin error aquí; id es never
```

Con `extends`, TypeScript avisa del conflicto al declarar. Con `&`, el conflicto queda oculto en un tipo con propiedad `never`. Además, `interface ... extends` permite nombrar la relación y suele dar mensajes de error más legibles. La intersección es más flexible: funciona con uniones y con cualquier tipo, no solo con objetos.

</details>

<details>
<summary>Utility types básicos para objetos</summary>

TypeScript incluye tipos derivados de uso frecuente:

```typescript
interface User { id: number; name: string; email: string }

type UserPreview = Pick<User, 'id' | 'name'>;    // solo esas propiedades
type UserWithoutId = Omit<User, 'id'>;           // todas menos id
type UserPatch = Partial<User>;                  // todas opcionales
type ReadonlyUser = Readonly<User>;              // todas readonly
type Ages = Record<string, number>;              // firma de índice
```

Se estudian con más detalle en [Advanced Object Types](08-Advanced%20Object%20Types.md).

</details>

-----

## En entrevista

### Respuesta corta (junior)

Un tipo personalizado es un tipo que defines tú para describir tus datos. Se crea con `type` o con `interface`, y puede tener propiedades opcionales (`?`) y de solo lectura (`readonly`). Sirve para nombrar una forma una vez, reutilizarla y que el compilador detecte errores. Los tipos desaparecen al compilar: no validan datos en ejecución.

### Respuesta ampliada (semi-senior)

* **Tipado estructural:** dos tipos son compatibles si tienen la forma requerida, sin importar el nombre. Por eso un alias no crea un tipo nuevo.
* **`type` frente a `interface`:** para objetos son casi intercambiables. `interface` admite declaration merging y `extends`; `type` admite uniones, tuplas, primitivos, tipos condicionales y mapped types.
* **Composición:** `extends` (interfaces) o `&` (intersección). `extends` detecta conflictos al declarar; `&` los convierte en `never`.
* **Modificadores:** `?` añade `undefined` al tipo; `readonly` es superficial y solo de compilación.
* **Índices:** `[k: string]: T` para claves dinámicas. Con `noUncheckedIndexedAccess`, el acceso devuelve `T | undefined`.
* **Enums:** generan código en ejecución; una unión de literales es una alternativa ligera.
* **Límite:** los tipos son solo de compilación; los datos externos requieren validación en ejecución.

### Preguntas frecuentes de seguimiento

**1. ¿Cuál es la diferencia entre `type` e `interface`?**
Para describir objetos casi ninguna. `interface` se puede fusionar y ampliar con `extends`; `type` puede nombrar cualquier tipo (uniones, tuplas, primitivos, funciones) y no se fusiona. Se puede usar cualquiera para objetos; lo importante es la coherencia.

**2. ¿Qué es el declaration merging?**
Es que dos `interface` con el mismo nombre en el mismo ámbito se combinan en una con todas las propiedades. Sirve para ampliar tipos existentes, como los de una librería. Con `type`, repetir el nombre es un error.

**3. ¿Qué diferencia hay entre `extends` y una intersección?**
Ambos combinan formas. Con `extends`, un conflicto de tipos en una propiedad es un error al declarar; con `&`, no hay error y la propiedad queda como `never`. La intersección además funciona con uniones y con tipos que no son objetos.

**4. ¿Cuándo uso cuál?**
`interface` para formas de objeto que pueden extenderse o que una clase implementa; `type` para uniones, tuplas, funciones, tipos derivados o intersecciones. Si dudas, sigue la convención del equipo.

**5. ¿`readonly` hace inmutable un objeto?**
No. Solo el compilador impide reasignar esa propiedad, y es superficial: un array o un objeto anidado se puede seguir mutando salvo que también sea `readonly`. En ejecución no hay protección.

**6. ¿Qué es una firma de índice y qué riesgo tiene?**
Es `[key: string]: T`, para objetos con claves dinámicas. El riesgo es que TypeScript asume que cualquier clave existe y devuelve `T`, aunque en ejecución sea `undefined`. `noUncheckedIndexedAccess` lo tipa como `T | undefined`.

-----

## Siguiente lección

Ahora que puedes definir tus propios tipos, el paso que sigue es combinar varios posibles en uno solo: [Union Types](06-Union%20Types.md).
