# Union Types: un valor que puede ser de varios tipos

## En una frase

Una **unión** (`A | B`) es un tipo que acepta valores de `A` **o** de `B`; TypeScript solo te deja usar lo que es seguro para todos los miembros, hasta que compruebas cuál es.

-----

## Antes de empezar

Conviene que ya sepas:

* Los tipos primitivos y los literales: [Types](01-Types.md).
* Cómo declarar tipos propios con `type`: [Custom Types](05-Custom%20Types.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Unión (union type):** tipo formado por varios **miembros**; un valor pertenece a al menos uno de ellos. Se escribe con `|`.
* **Miembro de la unión:** cada tipo que aparece en la unión.
* **Tipo literal:** tipo que representa un único valor concreto, como `'red'` o `404`.
* **Discriminante:** propiedad con tipo literal, presente en todos los miembros, que permite distinguirlos.
* **`never`:** tipo sin valores posibles. Aparece cuando TypeScript descarta todos los casos.

-----

## El problema

Un valor a veces puede tener más de un tipo. Por ejemplo, un identificador que llega como `string` o como `number`. Con `any` compila, pero desactiva la verificación:

```ts
let id: any = 1;
id.toUpperCase(); // compila, pero falla en ejecución: id es un número
```

Un solo tipo (`string`) es demasiado estricto; `any` es demasiado permisivo. La unión describe exactamente el punto medio: "uno de estos tipos, y ninguno más".

```
  string        number
 +--------+   +--------+
 |        |   |        |        string | number
 |  "a"   |   |   1    |   =   acepta "a" y 1,
 |  "b"   |   |   2    |       rechaza true, null, {}
 +--------+   +--------+
```

-----

## Cómo funciona

### Definir una unión

```ts
let id: string | number;

id = 1;      // válido
id = '001';  // válido
id = true;   // Error: Type 'boolean' is not assignable to type 'string | number'
```

Puede escribirse en cualquier posición donde vaya un tipo: variables, parámetros, retornos, propiedades.

```ts
function getMarginLeft(margin: string | number) {
  return { marginLeft: margin };
}
```

### Solo lo común a todos los miembros

Sobre un valor de tipo unión, TypeScript permite únicamente las propiedades y métodos que **existen en todos los miembros**. Es una regla de seguridad: no sabe cuál es el tipo real.

```ts
function format(value: string | number) {
  value.toString();  // válido: ambos tienen toString()
  value.toFixed(2);  // Error: Property 'toFixed' does not exist on type 'string'
}
```

Con objetos ocurre lo mismo:

```ts
type Goose = { isPettable: boolean; hasFeathers: boolean };
type Moose = { isPettable: boolean; hasHoofs: boolean };

function describe(animal: Goose | Moose) {
  animal.isPettable; // válido: está en ambos
  animal.hasHoofs;   // Error: Property 'hasHoofs' does not exist on type 'Goose | Moose'
}
```

### Acceder a lo específico: reducción de tipo (narrowing)

Para usar lo propio de un miembro, se comprueba primero cuál es. Dentro de esa comprobación, TypeScript **reduce** la unión a un solo miembro. Este mecanismo se llama *narrowing* y se estudia a fondo en [Type Narrowing](07-Type%20Narrowing.md). Aquí, lo mínimo:

```ts
function format(value: string | number) {
  if (typeof value === 'string') {
    return value.toUpperCase(); // aquí value es string
  }
  return value.toFixed(2);      // aquí value es number
}
```

Con objetos, el operador `in` comprueba si existe una propiedad:

```ts
function describe(animal: Goose | Moose) {
  if ('hasHoofs' in animal) {
    return 'Tiene pezuñas'; // animal es Moose
  }
  return 'Tiene plumas';    // animal es Goose
}
```

### Uniones de literales

Los miembros de una unión pueden ser valores concretos. Así se modelan conjuntos cerrados de opciones:

```ts
type Color = 'green' | 'yellow' | 'red';

function changeLight(color: Color) {
  // ...
}

changeLight('red');    // válido
changeLight('purple'); // Error: Argument of type '"purple"' is not assignable to parameter of type 'Color'
```

Es la alternativa habitual a un `string` genérico: el editor autocompleta las opciones y el compilador detecta errores de escritura.

### Inferencia de uniones en retornos

Si una función devuelve valores de tipos distintos, TypeScript infiere el retorno como unión:

```ts
type Book = { title: string };

function findBook(id: number) {
  if (id === 1) return { title: 'Clean Code' } as Book;
  return 'No encontrado';
}
// tipo de retorno inferido: "No encontrado" | Book (una unión con el literal)
```

### Uniones y arreglos

Los paréntesis cambian el significado:

```ts
const mixed: (string | number)[] = [1, 'a', 2]; // arreglo cuyos elementos son string o number
const either: string | number[] = 'a';          // un string, O un arreglo de solo números
```

`(string | number)[]` es "un arreglo de uniones"; `string | number[]` es "una unión entre un `string` y un arreglo de `number`".

### Unión discriminada

Cuando los miembros son objetos, se les añade una propiedad **literal** común (el discriminante). Comprobarla reduce la unión a un miembro concreto:

```ts
type Circle = { kind: 'circle'; radius: number };
type Square = { kind: 'square'; side: number };
type Shape = Circle | Square;

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2; // shape es Circle
    case 'square':
      return shape.side ** 2;             // shape es Square
  }
}
```

Es el patrón estándar para modelar estados: cada variante lleva solo los datos que le corresponden, y el compilador impide acceder a los de otra.

### Intersección (`&`) frente a unión (`|`)

Son operaciones opuestas:

* `A | B`: el valor es `A` **o** `B`. Solo se puede usar lo común.
* `A & B`: el valor es `A` **y** `B` a la vez. Tiene las propiedades de ambos.

```ts
type WithId = { id: number };
type WithName = { name: string };

type User = WithId & WithName;
const u: User = { id: 1, name: 'Ana' }; // debe tener ambas propiedades
```

Una intersección de primitivos incompatibles, como `string & number`, se reduce a `never`.

### `never` y comprobación de exhaustividad

En cada rama, TypeScript descarta los miembros ya tratados. Si se cubren todos, lo que queda es `never`. Se aprovecha para que el compilador avise cuando se añade una variante y no se maneja:

```ts
function assertNever(value: never): never {
  throw new Error(`Caso no manejado: ${JSON.stringify(value)}`);
}

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'square':
      return shape.side ** 2;
    default:
      return assertNever(shape); // shape es never mientras todos los casos estén cubiertos
  }
}
```

Si mañana se agrega `Triangle` a `Shape` y no se añade su `case`, `shape` en el `default` será `Triangle`, y la llamada a `assertNever` dará error de compilación: `Argument of type 'Triangle' is not assignable to parameter of type 'never'`.

-----

## Ejemplo completo

Estado de una petición, modelado con una unión discriminada. Es un patrón muy frecuente en componentes React:

```tsx
type RequestState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: string[] }
  | { status: 'error'; message: string };

function assertNever(value: never): never {
  throw new Error(`Estado no manejado: ${JSON.stringify(value)}`);
}

function render(state: RequestState): string {
  switch (state.status) {
    case 'idle':
      return 'Sin iniciar';
    case 'loading':
      return 'Cargando...';
    case 'success':
      return `Recibidos ${state.data.length} elementos`; // data existe solo aquí
    case 'error':
      return `Error: ${state.message}`;                  // message existe solo aquí
    default:
      return assertNever(state);
  }
}

render({ status: 'success', data: ['a', 'b'] }); // "Recibidos 2 elementos"
render({ status: 'error' });                     // Error: falta 'message'
```

Puntos clave:

1. Cada estado tiene solo los datos que le corresponden; no hay un `data` opcional que pueda existir a destiempo.
2. Comprobar `status` reduce la unión y desbloquea `data` o `message`.
3. `assertNever` convierte un estado olvidado en un error de compilación.

-----

## Errores comunes

### 1. Acceder a una propiedad que no es común

```ts
function len(x: string | number) {
  return x.length; // Error: Property 'length' does not exist on type 'number'
}
```

**Por qué pasa:** `x` podría ser un `number`, y `number` no tiene `length`.
**Solución:** reduce primero: `typeof x === 'string' ? x.length : String(x).length`.

### 2. Confundir `string | number[]` con `(string | number)[]`

**Por qué pasa:** sin paréntesis, el sufijo `[]` se aplica solo a `number`.
**Solución:** usa paréntesis cuando el arreglo contiene una unión.

### 3. Asignar un objeto incompleto a una unión de objetos

```ts
const a: Goose | Moose = { isPettable: true };
// Error: el objeto debe cumplir con Goose completo o con Moose completo
```

**Por qué pasa:** el valor debe ser asignable a *al menos un* miembro completo, no a la parte común.
**Solución:** proporciona todas las propiedades de un miembro.

### 4. Discriminante de tipo `string` en lugar de literal

```ts
type Bad = { kind: string; radius: number } | { kind: string; side: number };
```

**Por qué pasa:** con `string`, comprobar `kind` no distingue los miembros, así que no hay reducción.
**Solución:** usa literales (`'circle'`, `'square'`).

### 5. `switch` sin caso por defecto que compruebe exhaustividad

**Por qué pasa:** al añadir una variante nueva, el `switch` no la maneja y, si el tipo de retorno lo permite (por ejemplo, `void`, `undefined` o uno inferido), la función devuelve `undefined` o cae en otra rama sin que el compilador avise.
**Solución:** añade `default: return assertNever(x)`.

-----

## Cuándo sí y cuándo no

**Usa uniones para:**

* Valores que legítimamente pueden ser de varios tipos (`string | number`, `T | null`).
* Conjuntos cerrados de opciones (uniones de literales).
* Estados con datos distintos por variante (unión discriminada).

**Evita:**

* Uniones enormes de primitivos sin significado; suelen indicar un diseño confuso.
* Un solo objeto con muchas propiedades opcionales (`data?`, `error?`) cuando en realidad son estados excluyentes: una unión discriminada los hace imposibles de combinar mal.
* `any` cuando la unión describe el caso: `any` desactiva el chequeo, la unión lo conserva.

-----

## Resumen en 5 líneas

1. `A | B` acepta valores de `A` o de `B`; se escribe con `|`.
2. Solo se puede usar lo común a todos los miembros, hasta reducir con `typeof`, `in` o un discriminante.
3. Las uniones de literales (`'a' | 'b'`) modelan opciones cerradas.
4. La unión discriminada usa una propiedad literal común para distinguir variantes.
5. `never` en el `default` de un `switch` garantiza que todos los casos están cubiertos.

-----

## Para profundizar

<details>
<summary>Uniones y `null` / `undefined`</summary>

Con `strictNullChecks` (incluido en `strict`), `null` y `undefined` no son asignables a otros tipos. Para permitirlos se usa una unión: `string | null`. Es la base de tipos opcionales y de "todavía no hay dato", como `useState<User | null>(null)`.

</details>

<details>
<summary>Uniones con `as const` en lugar de `enum`</summary>

Un arreglo `as const` permite derivar la unión de literales sin repetirla:

```ts
const COLORS = ['green', 'yellow', 'red'] as const;
type Color = (typeof COLORS)[number]; // 'green' | 'yellow' | 'red'
```

Así el valor en tiempo de ejecución y el tipo siempre coinciden.

</details>

<details>
<summary>Distribución de tipos condicionales sobre uniones</summary>

Los tipos condicionales se **distribuyen** sobre las uniones: `T extends U ? X : Y` con `T = A | B` se evalúa por separado para `A` y para `B`, y el resultado es la unión. Así funcionan utilidades como `Exclude<T, U>` y `Extract<T, U>`: `Exclude<'a' | 'b' | 'c', 'a'>` resulta en `'b' | 'c'`.

</details>

-----

## En entrevista

### Respuesta corta (junior)

Una unión de tipos permite que un valor sea de uno de varios tipos, por ejemplo `string | number`. TypeScript solo deja usar lo común a todos los miembros; para usar algo específico se comprueba el tipo con `typeof` u otra guarda, y así se reduce la unión.

### Respuesta ampliada (semi-senior)

* **Seguridad frente a `any`:** la unión restringe los valores posibles y conserva el chequeo; `any` lo desactiva.
* **Reducción de tipo:** `typeof`, `instanceof`, `in`, igualdad y discriminantes refinan la unión dentro de cada rama.
* **Unión discriminada:** propiedad literal común en cada miembro. Evita estados imposibles (por ejemplo, `data` y `error` a la vez) mejor que propiedades opcionales.
* **Exhaustividad:** en el `default`, el valor restante tiene tipo `never`; asignarlo a `never` (`assertNever`) convierte una variante olvidada en error de compilación.
* **Unión frente a intersección:** `|` es "o" (menos propiedades utilizables); `&` es "y" (más propiedades). Una intersección de primitivos distintos da `never`.
* **Literales:** las uniones de literales reemplazan a `enum` en muchos casos, sin coste en tiempo de ejecución.

### Preguntas frecuentes de seguimiento

**1. ¿Por qué no puedo llamar a `toFixed` sobre `string | number`?**
Porque el valor podría ser un `string`, que no tiene ese método. TypeScript solo permite lo común a todos los miembros hasta que se reduzca el tipo.

**2. ¿Diferencia entre `string | number[]` y `(string | number)[]`?**
La primera es un `string` o un arreglo de números. La segunda es un arreglo cuyos elementos pueden ser `string` o `number`.

**3. ¿Qué es una unión discriminada?**
Una unión de objetos donde cada miembro tiene una propiedad con tipo literal distinto, como `kind: 'circle'`. Al comprobarla, TypeScript sabe qué miembro es y qué otras propiedades existen.

**4. ¿Para qué sirve `never` en un `switch`?**
Si todos los casos están cubiertos, el valor en el `default` es `never`. Asignarlo a un parámetro `never` hace que, al añadir una variante, el compilador marque el caso faltante.

**5. ¿Cuál es la diferencia entre `A | B` y `A & B`?**
La unión es "A o B" y expone solo lo común; la intersección es "A y B" y expone las propiedades de ambos.

**6. ¿Unión de literales o `enum`?**
La unión de literales no genera código en ejecución y se integra con inferencia y `as const`. El `enum` genera un objeto en tiempo de ejecución; conviene solo si se necesita ese objeto o su comportamiento inverso (en enums numéricos).

-----

## Siguiente lección

Ya sabes declarar uniones; ahora toca ver todas las formas de reducirlas con seguridad: [Type Narrowing](07-Type%20Narrowing.md).
