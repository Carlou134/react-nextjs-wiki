# Tipos básicos en TypeScript

## En una frase

TypeScript es JavaScript con un **sistema de tipos**: revisa tu código antes de ejecutarlo y avisa cuando un valor se usa de una forma que su tipo no permite.

-----

## Antes de empezar

Conviene que ya sepas:

* JavaScript básico: variables (`let`, `const`), funciones y objetos.
* Usar una terminal y tener Node.js instalado.

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Tipo:** conjunto de valores posibles y de operaciones válidas sobre ellos (`string`, `number`, `boolean`...).
* **Sistema de tipos:** las reglas con las que TypeScript decide si el código es coherente.
* **Inferencia de tipos:** TypeScript deduce el tipo a partir del valor asignado, sin que lo escribas.
* **Anotación de tipo:** tipo escrito de forma explícita después de `:`.
* **Transpilar / compilar:** convertir `.ts` en `.js`. Los tipos se eliminan en ese paso.
* **Superset (superconjunto):** todo JavaScript válido es TypeScript válido; TypeScript solo añade cosas.

-----

## El problema

JavaScript se diseñó en 1995 como un lenguaje flexible para scripts pequeños en el navegador. Esa flexibilidad se vuelve un costo en proyectos grandes: JavaScript acepta casi cualquier operación y los errores aparecen **en tiempo de ejecución**, cuando el usuario ya está usando la aplicación.

```js
let price = 10;
price = 'diez';          // JavaScript lo permite
price.toFixed(2);        // TypeError en ejecución: price.toFixed is not a function
```

Microsoft lanzó TypeScript en 2012 para detectar esa clase de errores **antes** de ejecutar: mientras escribes (en el editor) y al compilar.

Importante: los tipos existen **solo en tiempo de compilación**. TypeScript no valida datos en ejecución (por ejemplo, la respuesta de una API); esa parte sigue siendo tu responsabilidad.

-----

## Cómo funciona

### Del `.ts` al `.js`

1. Escribes código en archivos `.ts`.
2. El compilador de TypeScript (`tsc`) revisa los tipos y reporta errores.
3. Genera un archivo `.js` sin tipos, que es el que corre en Node o en el navegador.

Entrada TypeScript:

```ts
let firstName = 'Anders';
```

Salida JavaScript:

```js
let firstName = 'Anders';
```

Son iguales: el TypeScript de este ejemplo no tiene nada que eliminar. Las anotaciones de tipo desaparecen al compilar.

```
archivo.ts --(tsc: revisa tipos y elimina anotaciones)--> archivo.js --(Node / navegador)--> ejecución
```

### Ejecutar y compilar un archivo

Instala TypeScript (globalmente o, más habitual, como dependencia del proyecto):

```bash
npm install -g typescript
```

Compilar y ejecutar en dos pasos:

```bash
tsc archivo.ts      # genera archivo.js
node archivo.js
```

Ejecutar directo con **ts-node** (herramienta aparte que compila en memoria):

```bash
npm install -g ts-node
ts-node archivo.ts
```

Nota: si `tsc` encuentra errores de tipos, los muestra, pero por defecto **igual genera el `.js`** (la opción `noEmitOnError` cambia eso). Se configura en el [archivo tsconfig](02-Archivo%20tsconfig.md).

### Los tipos primitivos

TypeScript reconoce los primitivos de JavaScript:

| Tipo | Ejemplo | Nota |
|------|---------|------|
| `string` | `'hola'` | Texto |
| `number` | `42`, `3.14` | Enteros y decimales usan el mismo tipo |
| `boolean` | `true` | |
| `null` | `null` | Ausencia intencional de valor |
| `undefined` | `undefined` | Valor no asignado |
| `bigint` | `10n` | Enteros grandes |
| `symbol` | `Symbol('id')` | Identificadores únicos |

Los tipos van en minúscula. `String`, `Number` y `Boolean` (con mayúscula) son los objetos envoltorio de JavaScript y casi nunca son lo que quieres.

### Inferencia: TypeScript deduce el tipo

Cuando declaras una variable con valor inicial, TypeScript **infiere** su tipo y no permite asignarle otro:

```ts
let order = 'first';   // tipo inferido: string

order = 1;
// Error: Type 'number' is not assignable to type 'string'.
```

Correcto:

```ts
order = '1';           // sigue siendo string
```

La inferencia también distingue `let` de `const`:

```ts
let a = 'first';       // tipo: string (puede cambiar de valor, así que se "ensancha")
const b = 'first';     // tipo: 'first' (un literal: solo puede valer eso)
```

### Formas de los tipos (type shapes)

Cada tipo tiene propiedades y métodos conocidos. TypeScript los usa para avisarte de accesos que no existen:

```ts
'OH'.length;            // 2, válido
'MY'.toLowerCase();     // 'my', válido

'MY'.toLowercase();
// Error: Property 'toLowercase' does not exist on type '"MY"'.
// Did you mean 'toLowerCase'?
```

### Anotaciones de tipo

Una anotación se escribe con dos puntos y el tipo, justo después del nombre:

```ts
let mustBeAString: string;     // sin valor inicial, pero solo admite string
mustBeAString = 'Catdog';      // correcto

mustBeAString = 1337;
// Error: Type 'number' is not assignable to type 'string'.
```

Cuándo anotar y cuándo dejar que TypeScript infiera:

* **Deja inferir** cuando hay valor inicial obvio: `let count = 0;`.
* **Anota** cuando no hay valor inicial, en los **parámetros de funciones** y en valores que vienen de fuera (por ejemplo `const port: number = readPort();` si la función no declara su retorno).

### `any`: desactivar el sistema de tipos

`any` significa "cualquier cosa, no verifiques nada". Una variable declarada **sin valor inicial ni anotación** recibe `any` implícito:

```ts
let onOrOff;          // any implícito

onOrOff = 1;
onOrOff = false;      // sin error
```

Con la opción `noImplicitAny` (incluida en `strict`), TypeScript sigue el tipo por el flujo del código en casos como este, pero los **parámetros sin tipo** sí producen error. Además, `any` también se puede escribir a propósito:

```ts
let data: any = JSON.parse('{"a": 1}');   // JSON.parse devuelve any
data.no.existe.esto;                      // sin error de compilación; falla al ejecutar
```

`any` es contagioso: cualquier valor derivado de un `any` pierde la protección. Úsalo lo mínimo posible.

### `unknown`: la alternativa segura a `any`

`unknown` también acepta cualquier valor, pero **no permite usarlo** hasta que compruebes su tipo:

```ts
const value: unknown = JSON.parse('"hola"');

value.toUpperCase();
// Error: 'value' is of type 'unknown'.

if (typeof value === 'string') {
  value.toUpperCase();   // correcto: aquí TypeScript sabe que es string
}
```

Esa comprobación se llama *narrowing* (se ve en [Type Narrowing](07-Type%20Narrowing.md)). Regla práctica: si no conoces el tipo, usa `unknown`, no `any`.

### `void` y `never`

* **`void`:** tipo de retorno de una función que no devuelve un valor útil.
* **`never`:** tipo de algo que **nunca** produce un valor: una función que siempre lanza un error o entra en un bucle infinito.

```ts
function logMessage(message: string): void {
  console.log(message);          // no devuelve nada
}

function fail(message: string): never {
  throw new Error(message);      // nunca termina normalmente
}
```

Más sobre tipos de retorno en [Functions](03-Functions.md).

### `null` y `undefined` con `strict`

Con `strictNullChecks` activado (incluido en `"strict": true` de tu `tsconfig`), `null` y `undefined` **no** son asignables a otros tipos:

```ts
let title: string = null;
// Error: Type 'null' is not assignable to type 'string'.
```

Si un valor puede estar ausente, debes declararlo. Se usa una unión con `|` (más en [Union Types](06-Union%20Types.md)):

```ts
function shout(nickname: string | null) {   // puede ser string o null
  nickname.toUpperCase();
  // Error: 'nickname' is possibly 'null'.

  if (nickname !== null) {
    nickname.toUpperCase();                 // correcto
  }
}
```

Sin `strictNullChecks`, `null` y `undefined` se aceptan en todos los tipos y pierdes esa protección. Es la fuente clásica de `Cannot read properties of undefined`.

### Tipos literales

Un tipo puede ser un **valor exacto**, no solo una categoría:

```ts
let direction: 'left' | 'right';   // solo admite esos dos textos
direction = 'left';                // correcto
direction = 'up';
// Error: Type '"up"' is not assignable to type '"left" | "right"'.
```

Funcionan con `string`, `number` y `boolean` (`let ok: true`). Son la base de los [Union Types](06-Union%20Types.md).

### `as const`

`as const` le dice a TypeScript que trate un valor como **inmutable y lo más específico posible**:

```ts
const config = { mode: 'dark', retries: 3 };
// tipo: { mode: string; retries: number }

const strictConfig = { mode: 'dark', retries: 3 } as const;
// tipo: { readonly mode: 'dark'; readonly retries: 3 }

const roles = ['admin', 'user'] as const;
// tipo: readonly ['admin', 'user']  (tupla de solo lectura)
```

Es útil para derivar un tipo a partir de un valor:

```ts
type Role = (typeof roles)[number];   // 'admin' | 'user'
```

`as const` solo afecta al tipo; no congela el objeto en ejecución (para eso existe `Object.freeze`).

-----

## Ejemplo completo

```ts
// Constantes con tipos literales inferidos
const TAX_RATE = 0.16;                       // tipo: 0.16

// Función con parámetros anotados y retorno anotado
function totalWithTax(price: number, quantity: number): number {
  return price * quantity * (1 + TAX_RATE);
}

// Valor que puede no existir: null explícito
let coupon: string | null = null;

// Valor de origen desconocido: unknown + comprobación
function parseQuantity(input: unknown): number {
  if (typeof input === 'number') return input;
  if (typeof input === 'string') return Number(input);
  return 0;
}

const qty = parseQuantity('3');              // number
console.log(totalWithTax(10, qty));          // aprox. 34.8

coupon = 'PROMO10';
if (coupon !== null) {
  console.log(coupon.toLowerCase());         // narrowing: aquí es string
}

totalWithTax('10', 2);
// Error: Argument of type 'string' is not assignable to parameter of type 'number'.
```

Puntos clave:

1. Los parámetros se anotan; el tipo de `qty` y de `TAX_RATE` se infiere.
2. `unknown` obliga a comprobar el tipo antes de usar el valor.
3. `string | null` hace explícito que el valor puede faltar.
4. El último error se detecta al compilar, no al ejecutar.

-----

## Errores comunes

### 1. Anotar todo, incluso lo obvio

```ts
const count: number = 0;   // redundante
```

**Por qué pasa:** se cree que más anotaciones equivalen a más seguridad.
**Cómo se arregla:** deja inferir cuando hay valor inicial (`const count = 0;`) y anota parámetros, retornos públicos y valores sin inicializar.

### 2. Usar `any` para "que compile"

```ts
function parse(data: any) {
  return data.user.name;   // sin verificación
}
```

**Por qué pasa:** `any` silencia el error en lugar de resolverlo.
**Cómo se arregla:** usa `unknown` y comprueba el tipo, o define el tipo real de `data` (ver [Advanced Object Types](08-Advanced%20Object%20Types.md)).

### 3. Creer que los tipos validan en ejecución

```ts
const user = JSON.parse(text) as { name: string };   // TypeScript confía; no valida
```

**Por qué pasa:** los tipos se borran al compilar; `as` solo cambia lo que TypeScript cree, no lo que hay en memoria.
**Cómo se arregla:** valida los datos externos en ejecución (comprobaciones manuales o una librería de validación) antes de tratarlos como un tipo.

### 4. Desactivar `strict` por comodidad

**Por qué pasa:** los errores de null y de `any` implícito parecen molestos.
**Cómo se arregla:** mantén `"strict": true` en el [tsconfig](02-Archivo%20tsconfig.md). Esos errores señalan fallos reales.

### 5. Confundir `let` y `const` en la inferencia

```ts
let mode = 'dark';
const setMode = (m: 'dark' | 'light') => {};
setMode(mode);
// Error: Argument of type 'string' is not assignable to parameter of type '"dark" | "light"'.
```

**Por qué pasa:** con `let` el tipo se ensancha a `string`.
**Cómo se arregla:** usa `const mode = 'dark';`, o anota `let mode: 'dark' | 'light' = 'dark';`.

-----

## Cuándo sí y cuándo no

**Anota explícitamente cuando:**

* Declaras una variable sin valor inicial.
* Defines parámetros de funciones (siempre).
* Quieres un tipo más estrecho o más amplio que el inferido.

**Deja que TypeScript infiera cuando:**

* Hay un valor inicial claro (`let total = 0;`).
* Es una variable local cuyo tipo es evidente por la expresión.

**Usa cada "tipo especial" así:**

* `unknown`: datos de tipo incierto que vas a comprobar.
* `any`: solo como salida temporal, al migrar código JavaScript.
* `void`: funciones sin valor de retorno útil.
* `never`: funciones que no terminan y comprobaciones de exhaustividad.

-----

## Resumen en 5 líneas

1. TypeScript añade un sistema de tipos a JavaScript; los tipos se eliminan al compilar y no validan en ejecución.
2. Con valor inicial, el tipo se **infiere**; sin él (o en parámetros), se **anota** con `: tipo`.
3. `any` desactiva la verificación; prefiere `unknown` y comprueba el tipo antes de usarlo.
4. Con `strict` (`strictNullChecks`), `null` y `undefined` no entran en otros tipos: decláralos con `string | null`.
5. Los tipos literales y `as const` estrechan un valor a su forma exacta.

-----

## Para profundizar

<details>
<summary>Ensanchamiento (widening) de literales</summary>

Con `const`, el valor no puede cambiar, así que TypeScript infiere el literal (`'first'`). Con `let` puede cambiar, así que infiere el tipo general (`string`). En propiedades de objetos, el tipo también se ensancha, aunque el objeto sea `const`:

```ts
const user = { role: 'admin' };   // { role: string }
```

`as const` evita ese ensanchamiento.

</details>

<details>
<summary>Comprobación de exhaustividad con never</summary>

`never` sirve para que el compilador avise si olvidas un caso:

```ts
type Light = 'red' | 'green';

function action(light: Light): string {
  switch (light) {
    case 'red':
      return 'stop';
    case 'green':
      return 'go';
    default: {
      const unreachable: never = light;   // error si se añade un caso a Light y no se maneja aquí
      return unreachable;
    }
  }
}
```

Si añades `'yellow'` a `Light` y no lo manejas, el `default` deja de ser inalcanzable y TypeScript marca error.

</details>

<details>
<summary>void frente a undefined</summary>

`void` indica "no uses el valor de retorno"; `undefined` es un valor concreto. Una función tipada con retorno `void` puede, en ciertos contextos (como callbacks), devolver otra cosa sin error; el tipo solo dice que quien llama no debe usar ese valor. Por eso `[1, 2].forEach(n => list.push(n))` compila aunque `push` devuelva un número.

</details>

<details>
<summary>Qué incluye "strict"</summary>

`"strict": true` activa varias comprobaciones a la vez, entre ellas `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes` y `strictBindCallApply`. Desde TypeScript 6.0 el valor por defecto de `strict` es `true`; en versiones anteriores era `false`. Con `strict` puedes desactivar una en particular si es imprescindible, pero lo recomendable es partir con todas activas. Más en [Archivo tsconfig](02-Archivo%20tsconfig.md).

</details>

<details>
<summary>Herramientas de ejecución</summary>

`tsc` compila; `ts-node` ejecuta compilando en memoria. Los proyectos de React y Next.js suelen delegar la transformación de `.ts`/`.tsx` al bundler o al framework, y usan `tsc --noEmit` solo para comprobar tipos.

</details>

-----

## En entrevista

### Respuesta corta (junior)

TypeScript es un superset de JavaScript que añade tipado estático. Detecta errores de tipos al compilar, antes de ejecutar, y se convierte a JavaScript normal. Infiere el tipo cuando asignas un valor inicial y permite anotarlo con `: tipo` cuando hace falta. Los tipos básicos son `string`, `number` y `boolean`.

### Respuesta ampliada (semi-senior)

* **Tipado estático, borrado en compilación:** los tipos no existen en ejecución; no validan datos externos. Para eso hace falta validación en runtime.
* **Inferencia:** reduce el ruido; se anota en parámetros, variables sin valor inicial y fronteras de módulo (retornos públicos).
* **`let` vs `const`:** `const` infiere literales; `let` ensancha al tipo general. `as const` fija literales y `readonly`.
* **`any` vs `unknown`:** `any` desactiva el chequeo y contagia; `unknown` obliga a estrechar el tipo antes de usarlo.
* **`strictNullChecks`:** `null` y `undefined` se tratan como tipos propios; hay que declarar la ausencia con uniones y estrecharla.
* **`never`:** tipo bottom, sin valores; sirve para funciones que no retornan y para comprobar exhaustividad.
* **Trade-off:** `strict` da más errores al inicio, pero previene fallos en producción y facilita refactorizar.

### Preguntas frecuentes de seguimiento

**1. ¿Cuál es la diferencia entre `any` y `unknown`?**
Ambos aceptan cualquier valor. Con `any` puedes hacer cualquier operación sin verificación; con `unknown` debes comprobar el tipo antes de usarlo, por lo que es más seguro.

**2. ¿Cuándo anotas y cuándo dejas inferir?**
Anoto parámetros, variables sin valor inicial y retornos de funciones expuestas. Dejo inferir en variables locales con valor inicial evidente.

**3. ¿Qué hace `strictNullChecks`?**
Impide asignar `null` o `undefined` a tipos que no los incluyan. Hay que declarar la ausencia (`string | null`) y comprobarla antes de usar el valor.

**4. ¿Los tipos de TypeScript existen en ejecución?**
No. Se borran al compilar. Por eso una respuesta de API tipada con una interfaz no está validada: TypeScript solo confía en lo que declaras.

**5. ¿Qué hace `as const`?**
Infiere el tipo más específico posible y marca las propiedades como `readonly`; los arrays pasan a ser tuplas de solo lectura. Es solo del sistema de tipos, no congela el objeto en ejecución.

**6. ¿Para qué sirve `never`?**
Representa valores que nunca ocurren: funciones que siempre lanzan error o no terminan. También permite forzar al compilador a avisar cuando una unión no se maneja por completo.

-----

## Siguiente lección

Ahora que conoces los tipos básicos, el paso que sigue es configurar cómo el compilador los revisa: [Archivo tsconfig](02-Archivo%20tsconfig.md).
