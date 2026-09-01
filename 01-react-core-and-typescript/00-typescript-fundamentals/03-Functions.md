# Functions

## Introduction

**Cuando declaramos una función en JavaScript, a menudo esperamos que se invoque con argumentos de un tipo determinado.** JavaScript no comparte nuestras expectativas: su flexibilidad de tipos a menudo permite que las **funciones** sean invocadas con tipos de argumentos inesperados. Incluso cuando esto no da lugar a errores, puede haber consecuencias negativas:

```js
function printLengthOfText(text) {
  console.log(text.length);
}

printLengthOfText(3); // Imprime: undefined
```

Los desarrolladores de JavaScript han encontrado soluciones para el manejo de errores para evitar estos efectos indeseables, pero estas técnicas pueden ser complicadas:

```js
function printLengthOfText(text) {
  if (typeof text !== 'string') {
    throw new Error('¡El argumento no es una cadena de texto!');
  }

  console.log(text.length);
}

printLengthOfText(3); // Error: ¡El argumento no es una cadena de texto!
```

En el código anterior, la función primero verifica el tipo del argumento. Si no coincide con el tipo esperado, se lanza un error; de lo contrario, continúa con su operación prevista.

Antes de explorar cómo TypeScript maneja este problema, ¡vamos a practicar corrigiendo algunos errores relacionados con tipos en JavaScript! Este es el tipo de problema que TypeScript nos ayuda a diagnosticar de manera temprana, antes de que se vuelva difícil de detectar.

---

## Parameter Type Annotations

**En TypeScript, los parámetros de las funciones pueden tener anotaciones de tipo** con la misma sintaxis que las declaraciones de variables: un colon al lado del nombre. Las anotaciones de tipo aseguran que los parámetros sean del tipo correcto:

```ts
function greet(name: string) {
  console.log(`Hello, ${name}!`);
}

greet('Katz'); // Imprime: Hello, Katz  

greet(1337); // Error: El argumento '1337' no es asignable al parámetro de tipo 'string'
```

Al colocar `: string` después del parámetro `name`, estamos declarando que `name` es de tipo `string`. Cualquier invocación de `greet()` debe pasar un valor de tipo `string` como primer argumento, o de lo contrario se lanzará un error.

Los parámetros para los cuales no proporcionamos anotaciones de tipo se asumen como de tipo `any`, de la misma manera que las **variables**.

```ts
function printKeyValue(key: string, value) {
  console.log(`${key}: ${value}`);
}

printKeyValue('Courage', 1337); // Imprime: Courage: 1337
printKeyValue('Mood', 'scared'); // Imprime: Mood: scared
```

Aquí, el parámetro `value` es una variable de tipo `any`: es compatible con cualquier tipo.

---

## Optional Parameters

**TypeScript normalmente da un error si no proporcionamos un valor para todos los argumentos de una función.** Esto no siempre es deseable; a veces queremos omitir la provisión de valores.

```ts
function greet(name: string) {
  console.log(`Hello, ${name || 'Anonymous'}!`);
}

greet('Anders'); // Imprime: Hello, Anders!
greet(); // Error de TypeScript: Se esperaban 1 argumento, pero se recibieron 0.
```

Cuando el fragmento de código anterior se compila a JavaScript, la función `greet()` imprimirá correctamente `'Hello, Anonymous!'`. Esto se debe a que cuando no se pasan argumentos, `name` tiene el valor *falsy* `undefined`, lo que significa que `name || 'Anonymous'` se evalúa como `'Anonymous'`. Dado que el código final funciona como se espera, queremos evitar que TypeScript lance errores.

Para indicar que un parámetro es intencionalmente opcional, agregamos un `?` después de su nombre. Esto le indica a TypeScript que el parámetro puede ser `undefined` y no siempre tiene que ser proporcionado.

```ts
function greet(name?: string) {
  console.log(`Hello, ${name || 'Anonymous'}!`);
}

greet(); // Imprime: Hello, Anonymous!
```

---

## Default Parameters

**Si un parámetro se le asigna un valor por defecto, TypeScript inferirá que el tipo de la variable es el mismo que el tipo del valor por defecto.** (Esto es similar a cómo TypeScript infiere el tipo de una variable inicializada para que sea el mismo que el tipo de su valor inicial).

El siguiente fragmento de código muestra una cadena para saludar al nombre de un usuario y usa el nombre `'Anonymous'` por defecto si no se proporciona ningún nombre.

```ts
function greet(name = 'Anonymous') {
  console.log(`Hello, ${name}!`);
}
```

La función `greet()` puede recibir una cadena o `undefined` como parámetro `name`; si se pasa cualquier otro tipo como argumento, TypeScript considerará eso un error de tipo.

Esta es una forma más limpia de obtener la misma funcionalidad que tuvimos en el ejercicio anterior. Allí, usamos `?` para marcar el parámetro `name` como opcional. Pero los parámetros con valores por defecto no necesitan un `?` después de su nombre, ya que asignar un valor por defecto ya implica que son parámetros opcionales.

---

## Inferring Return Types

**TypeScript también puede inferir los tipos de los valores devueltos por las funciones.** Lo hace observando los tipos de los valores después de las sentencias `return` de una función.

```ts
function createGreeting(name: string) {
  return `Hello, ${name}!`;
}

const myGreeting = createGreeting('Aisle Nevertell');
```

Así es como TypeScript puede inferir que `myGreeting` es de tipo `string`:

* La sentencia `return` de `createGreeting()` está seguida de una variable de tipo `string`, por lo que `createGreeting()` se infiere que devuelve un `string`.
* Al llamar `createGreeting('Aisle Nevertell')`, el valor devuelto debe ser de tipo `string`.
* `myGreeting` se inicializa con `createGreeting('Aisle Nevertell')`, que es un valor de tipo `string`.

¡Genial! Veamos cómo esto puede ayudarnos a corregir errores:

```ts
function ouncesToCups(ounces: number) {
  return `${ounces / 16} cups`;
}

const liquidAmount: number = ouncesToCups(3);
// Error: Type 'string' is not assignable to type 'number'.
```

Aquí, TypeScript pudo inferir que a `liquidAmount` se le estaba asignando un valor de tipo `string`, a pesar de que se había declarado como una variable de tipo `number`. Esto da como resultado correctamente un error.

---

## Explicit Return Types

**Si deseamos ser explícitos acerca de qué tipo devuelve una función, podemos agregar una anotación de tipo explícita después de su paréntesis de cierre.** Aquí, usamos la misma sintaxis que otras anotaciones de tipo: un colon seguido del tipo. TypeScript producirá un error para cualquier sentencia `return` en esa función que no devuelva el tipo correcto de valor.

```ts
function createGreeting(name?: string): string {
  if (name) {
    return `Hello, ${name}!`;
  }

  return undefined;
  // Error de TypeScript: El tipo 'undefined' no es asignable al tipo 'string'.
};
```

También podemos declarar explícitamente los tipos de retorno para las **funciones de flecha** (que fueron definidas en la versión ES6/ES2015 de JavaScript). Veremos los mismos tipos de mensajes de error para ambos tipos de función.

```ts
const createArrowGreeting = (name?: string): string => {
  if (name) {
    return `Hello, ${name}!`;
  }

  return undefined;
  // Error de TypeScript: El tipo 'undefined' no es asignable al tipo 'string'.
};
```

Ahora echemos un vistazo a un ejemplo de cómo la anotación explícita de tipos de retorno puede ser útil cuando estamos trabajando con el código de otras personas.

---

## Void Return Type

**Hasta ahora, hemos presentado un caso bastante convincente de que las anotaciones de tipo son muy útiles y deberían ser siempre usadas, a menos que haya una muy buena razón para no hacerlo.** Ayudan a mantener todo ordenado y facilitan la comprensión de nuestro código.

Por estas razones, a menudo se prefiere usar anotaciones de tipo para **funciones**, incluso cuando esas funciones no devuelven nada. Ejemplo:

```ts
function logGreeting(name: string) {
  console.log(`Hello, ${name}!`);
}
```

La función `logGreeting()` simplemente muestra un saludo en la consola. No hay un valor de retorno, por lo que debemos tratar el tipo de retorno como `void`. Una anotación de tipo adecuada para esta función sería:

```ts
function logGreeting(name: string): void {
  console.log(`Hello, ${name}!`);
}
```

---

## Documenting Functions

**TypeScript reconoce la sintaxis de comentarios de JavaScript:**

```js
// Este es un comentario de una sola línea
```

```js
/*
Este es un 
comentario
multilínea
*/
```

Sin embargo, es común en TypeScript ver un tercer estilo de comentario: **comentarios de documentación**. Un comentario de documentación se denota con la primera línea `/**` y la línea final `*/`. Es común que cada línea dentro del comentario comience con un asterisco (`*`):

```ts
/**
* Este es un comentario de documentación
*/
```

Los comentarios de documentación son especialmente útiles para documentar **funciones**. Colocamos el comentario de documentación de una función directamente sobre la declaración de la función. Podemos usar etiquetas especiales dentro del comentario para resaltar ciertos aspectos de la función. Podemos usar `@param` para describir cada uno de los parámetros de la función y `@returns` para describir lo que la función devuelve:

```ts
/**
 * Devuelve la suma de dos números.
 *
 * @param x - El primer número de entrada
 * @param y - El segundo número de entrada
 * @returns La suma de `x` y `y`
 *
 */
function getSum(x: number, y: number): number {
  return x + y;
}
```

Muchos editores de texto mostrarán útiles los comentarios de documentación, por ejemplo, al pasar el cursor sobre el nombre de una función.

---

## Review

**Aprendimos todo sobre cómo usar TypeScript para especificar los tipos de los parámetros de las funciones y los tipos de retorno. Ahora sabemos cómo:**

* Dar **anotaciones de tipo** a los parámetros de las funciones.
* Manejar las anotaciones de tipo para parámetros opcionales, que pueden tener valores predeterminados.
* Entender cómo TypeScript determina el tipo de retorno de una función.
* Especificar explícitamente los tipos de retorno para **funciones**, incluidas las funciones que no devuelven nada.
* Usar las habilidades anteriores para diagnosticar y corregir errores en nuestro código.

¡Buen trabajo!

---

