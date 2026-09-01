# Custom Types

## Introduction

¡Hasta ahora, has cubierto mucho de TypeScript! Entiendes todos los tipos que TypeScript define para ti: tipos primitivos y **arrays**. ¡Eso es un gran logro! Pero no te confíes demasiado, porque TypeScript también se puede usar para crear tipos personalizados, en lugar de estar limitado a los tipos predefinidos. Los tipos personalizados son lo que hacen a TypeScript realmente divertido y útil, ya que permiten una comprobación de tipos que está adaptada a tus propósitos exactos.

De hecho, ya has estudiado un ejemplo de un tipo personalizado: **tuplas**. Por ejemplo, el tipo de tupla `[string, string, number, boolean]` es un tipo personalizado que se puede usar con datos sobre los usuarios de un sitio web: nombre (string), apellido (string), edad (number) y si tienen una cuenta de pago (boolean).

Los tipos predefinidos son como ingredientes: pueden usarse por sí solos. A veces solo necesitas un string simple y a veces solo quieres comer un pepinillo. Sin embargo, los tipos predefinidos también pueden combinarse en tipos personalizados. Los tipos personalizados son como comidas completamente preparadas (pepinos, así como queso, pan y carne de hamburguesa).

Los tipos complejos que cubrimos aquí se pueden usar de la misma manera que los tipos más simples cubiertos antes. Pueden usarse como **anotaciones de tipo** durante la declaración de variables:

```typescript
let myVar: compType;
```

Y también se pueden usar como anotaciones de tipo para funciones:

```typescript
function testFn(param: compType): returnedCompType {
  /* Cuerpo de la función */
}
```

E incluso puedes hacer inferencia de tipos con tipos complejos:

```typescript
let inferredTypeVariable = testFn(myVar);
// La variable inferredTypeVariable tendrá el tipo returnedCompType.
```

Así que, sin más preámbulos, ¡vamos a sumergirnos en nuestro primer tipo complejo!

----

## Enums

Nuestro primer ejemplo de un tipo complejo también es uno de los más útiles: los **enums**. Usamos enums cuando queremos enumerar todos los valores posibles que una variable podría tener. Esto contrasta con la mayoría de los otros tipos que hemos estudiado. Una variable de tipo string puede tener cualquier cadena como valor; hay infinitas cadenas posibles, y sería imposible listarlas todas. De manera similar, una variable de tipo `boolean[]` puede tener cualquier arreglo de booleanos como valor; nuevamente, las posibilidades son infinitas.

```typescript
enum Direction {
  North,
  South,
  East,
  West
}
```

Hay muchas situaciones en las que podríamos querer limitar los valores posibles de una variable. Por ejemplo, el código anterior define el **enum** `Direction`, que representa las cuatro direcciones del compás: `Direction.North`, `Direction.South`, `Direction.East` y `Direction.West`. Cualquier otro valor, como `Direction.Southeast`, no está permitido. Mira el siguiente ejemplo:

```typescript
let whichWayToArcticOcean: Direction;
whichWayToArcticOcean = Direction.North; // No hay error de tipo.
whichWayToArcticOcean = Direction.Southeast; // Error de tipo: Southeast no es un valor válido para el enum Direction.
whichWayToArcticOcean = West; // Sintaxis incorrecta, debemos usar Direction.West en su lugar.
```

Como se muestra arriba, un tipo de **enum** se puede usar en una anotación de tipo como cualquier otro tipo.

Bajo el capó, TypeScript procesa estos tipos de **enum** utilizando números. Los valores de los **enum** se asignan a un valor numérico según su orden en la lista. El primer valor se asigna el número 0, el segundo el número 1, y así sucesivamente.

Por ejemplo, si establecemos `whichWayToArticOcean = Direction.North`, entonces `whichWayToArticOcean == 0` evaluará como verdadero. Además, podemos reasignar `whichWayToArticOcean` a un valor numérico, como `whichWayToArticOcean = 2`, y no se generará un error de tipo. Esto es porque `Direction.North`, `Direction.South`, `Direction.East` y `Direction.West` son iguales a 0, 1, 2 y 3, respectivamente.

Podemos cambiar el número inicial, escribiendo algo como esto:

```typescript
enum Direction {
  North = 7,
  South,
  East,
  West
}
```

Aquí, `Direction.North`, `Direction.South`, `Direction.East` y `Direction.West` son iguales a 7, 8, 9 y 10, respectivamente.

También podemos especificar todos los números por separado, si es necesario:

```typescript
enum Direction {
  North = 8,
  South = 2,
  East = 6,
  West = 4
}
```

(Estos números coinciden con las teclas del teclado numérico de muchos teclados).

Ahora, ¡vamos a practicar con los **enums** de TypeScript!

----

## String Enums vs. Numeric Enums

Los **enums** que hemos estudiado hasta ahora se conocen como **enums numéricos**, ya que están basados en números. TypeScript también nos permite usar **enums** basados en cadenas de texto, conocidos como **enums de cadenas**. Se definen de manera muy similar:

```typescript
enum DirectionNumber { North, South, East, West }
enum DirectionString { North = 'NORTH', South = 'SOUTH', East = 'EAST', West = 'WEST' }
```

Con los **enums numéricos**, los números pueden ser asignados automáticamente, pero con los **enums de cadenas**, debemos escribir explícitamente la cadena, como se muestra arriba. Técnicamente, cualquier cadena servirá: `North = 'JabberWocky'` es una definición de valor válida. Sin embargo, es mucho mejor usar la convención que se muestra aquí (`North = 'NORTH'`), donde el valor de la cadena de la variable del **enum** es simplemente la forma en mayúsculas del nombre de la variable. De esta forma, los mensajes de error y los registros serán mucho más informativos.

Recomendamos usar siempre **enums de cadenas** porque los **enums numéricos** permiten algunos comportamientos que pueden permitir que errores se cuelen en nuestro código. Por ejemplo, se pueden asignar números directamente a las variables de **enum** numérico:

```typescript
let whichWayToAntarctica: DirectionNumber;
whichWayToAntarctica = 1; // Código válido en TypeScript.
whichWayToAntarctica = DirectionNumber.South; // Válido, equivalente a la línea anterior.
```

Curiosamente, incluso asignar números arbitrarios, como `whichWayToAntarctica = 943205`, no generará errores de tipo.

Los **enums de cadenas** son mucho más estrictos. Con los **enums de cadenas**, ¡no se puede asignar cadenas arbitrarias a las variables!

```typescript
let whichWayToAntarctica: DirectionString;
whichWayToAntarctica = '\ (•◡•) / Arbitrary String \ (•◡•) /'; // ¡Error de tipo!
whichWayToAntarctica = 'SOUTH'; // ¡AÚN un error de tipo!
whichWayToAntarctica = DirectionString.South; // La única forma permitida de hacerlo.
```

Ahora, ¡vamos a practicar!

----

## Object Types

¡Es hora! Finalmente podemos hablar sobre la programación orientada a objetos y cómo se relaciona con TypeScript. Los **tipos de objetos** de TypeScript son extremadamente útiles, ya que nos permiten tener un control muy detallado sobre los tipos de variables en nuestros programas. También son los tipos personalizados más comunes, por lo que debemos entenderlos si queremos leer los programas de otras personas.

Aquí tienes una **anotación de tipo** para un objeto que representa a una persona:

```typescript
let aPerson: {name: string, age: number};
```

La **anotación de tipo** se parece a un **literal de objeto**, pero en lugar de valores después de las propiedades, tenemos tipos. Observa que la variable `aPerson` aún no ha sido asignada a un valor. Intentar asignar un valor a `aPerson` que no tenga las propiedades `name` y `age` con los tipos especificados generará un error de tipo:

```typescript
aPerson = {name: 'Aisle Nevertell', age: "wouldn't you like to know"}; // Error de tipo: la propiedad "age" tiene el tipo incorrecto.
aPerson = {name: 'Kushim', yearsOld: 5000}; // Error de tipo: no hay propiedad "age".
aPerson = {name: 'User McCodecad', age: 22}; // Código válido.
```

En el caso de **Kushim** arriba, el objeto tenía propiedades del tipo correcto. Sin embargo, se lanzó un error de tipo porque las propiedades no tenían los nombres correctos.

TypeScript no pone restricciones sobre los tipos de las propiedades de un objeto. ¡Pueden ser **enums**, **arrays** e incluso otros tipos de objetos!

```typescript
let aCompany: {
  companyName: string, 
  boss: {name: string, age: number}, 
  employees: {name: string, age: number}[], 
  employeeOfTheMonth: {name: string, age: number},  
  moneyEarned: number
};
```

Esto es solo una introducción a los **tipos de objetos** de TypeScript. Una descripción completa merecería una lección por sí sola (lo cual pronto exploraremos si seguimos aprendiendo). Por ahora, ¡practiquemos los conceptos básicos un poco más!

----

## Type Aliases

Una excelente manera de personalizar los tipos en nuestros programas es utilizar **alias de tipos**. Estos son nombres alternativos de tipo que elegimos por conveniencia. Usamos el formato `type <nombre del alias> = <tipo>`:

```typescript
type MyString = string;
let myVar: MyString = 'Hi'; // Código válido.
```

Crear nombres alternativos para `string` puede no ser muy útil, pero esto se puede hacer con cualquier tipo. Los alias de tipos son realmente útiles para referirse a tipos complicados que necesitan ser repetidos, especialmente **tipos de objetos** y **tipos de tuplas**. Recordemos el ejemplo de la empresa que vimos antes:

```typescript
let aCompany: { 
  companyName: string, 
  boss: { name: string, age: number }, 
  employees: { name: string, age: number }[], 
  employeeOfTheMonth: { name: string, age: number },  
  moneyEarned: number
};
```

¡Aquí hay una gran repetición innecesaria! (Y cuantas más veces repitamos algo, más oportunidades hay de cometer errores tipográficos). Esto se puede simplificar con **alias de tipos**:

```typescript
type Person = { name: string, age: number };
let aCompany: {
  companyName: string, 
  boss: Person, 
  employees: Person[], 
  employeeOfTheMonth: Person,  
  moneyEarned: number
};
```

Todo el mundo conoce la famosa cita de Shakespeare: "¿Qué hay en un nombre? Lo que llamamos una cadena, con cualquier otro nombre, olería igual de dulce". Los alias de TypeScript no son más que nombres. No tienen absolutamente ninguna influencia sobre cómo funcionan los tipos. Por ejemplo, el siguiente código no genera errores de tipo:

```typescript
type MyString = string; 
type MyOtherString = string;
let firstString: MyString = 'test';
let secondString: MyOtherString = firstString; // Código válido.
```

La razón por la que esto funciona es que `MyString` y `MyOtherString` no son tipos distintos. Son solo nombres alternativos para lo mismo.

Usando **alias de tipos**, podemos hacer que nuestro código sea mucho más fácil de entender. ¡Vamos a probarlo!

----

## Function Types

Una de las cosas interesantes de JavaScript es que **las funciones** se pueden asignar a **variables**.

```typescript
let myFavoriteFunction = console.log; // Fíjate en la ausencia de paréntesis.
myFavoriteFunction('Hello World'); // Imprime: Hello World
```

Una de las cosas interesantes de TypeScript es que podemos controlar con precisión qué tipos de funciones se pueden asignar a una variable. Hacemos esto utilizando **tipos de funciones**, que especifican los tipos de los argumentos y el tipo de retorno de una función. Aquí hay un ejemplo de un tipo de función que solo es compatible con funciones que reciben dos argumentos de tipo `string` y retornan un número:

```typescript
type StringsToNumberFunction = (arg0: string, arg1: string) => number;
```

Esta sintaxis es similar a la notación de flecha para funciones, excepto que en lugar del valor de retorno, ponemos el tipo de retorno. En este caso, el tipo de retorno es `number`. Como esto es solo un tipo, no escribimos el cuerpo de la función en absoluto. Una variable de tipo `StringsToNumberFunction` puede ser asignada a cualquier función compatible:

```typescript
let myFunc: StringsToNumberFunction;
myFunc = function(firstName: string, lastName: string) {
  return firstName.length + lastName.length;
};

myFunc = function(whatever: string, blah: string) {
  return whatever.length - blah.length;
};
// Ninguna de estas asignaciones genera un error de tipo.
```

Como vemos arriba, no importa cómo nombremos los parámetros de la función, siempre y cuando tengan los tipos correctos (`string` y `string`). Por lo tanto, no importa qué nombres les pongamos a los parámetros en la anotación de tipo (arriba elegimos `arg0` y `arg1`).

Hay algo importante que recordar aquí. ¡Nunca debemos caer en la tentación de omitir los nombres de los parámetros o los paréntesis alrededor de los parámetros en una anotación de tipo de función, incluso si solo hay un parámetro! Este código **no funcionará**:

```typescript
type StringToNumberFunction = (string) => number; // NO
type StringToNumberFunction = arg: string => number; // NO NO NO NO
```

Los **tipos de función** son más útiles cuando se aplican a **funciones de retorno (callback)**. Como las funciones de retorno son tan comunes, es útil saber cómo tiparlas correctamente. ¡Vamos a practicar el uso de los tipos de función con funciones de retorno!

----

## Generic Types

Los **genéricos** de TypeScript son una forma de crear colecciones de tipos (y funciones tipadas, entre otras cosas) que comparten ciertas similitudes formales. Estas colecciones están parametrizadas por una o más variables de tipo. Ahora que hemos aclarado esto, ¡pasemos a la revisión!

Hmm, quizás deberíamos discutir esto con un poco más de detalle. De hecho, ya hemos visto un ejemplo de un tipo genérico en uso. ¿Recuerdas la sintaxis de tipo de array `Array<T>`? Esto es genérico porque podemos sustituir cualquier tipo (ya sea predefinido o personalizado) en lugar de T. Por ejemplo, `Array<string>` es un array de cadenas de texto.

Los genéricos nos dan el poder de definir nuestras propias colecciones de tipos de objetos. Aquí tienes un ejemplo:

```typescript
type Family<T> = {
  parents: [T, T], mate: T, children: T[]
};
```

Este código define una colección de tipos de objetos, con un tipo diferente para cada valor de T. El genérico `Family<T>` no puede ser usado directamente como un tipo en una anotación de tipo. En su lugar, debemos sustituir T por algún tipo de nuestra elección, por ejemplo, `string`. Entonces, `Family<string>` es exactamente lo mismo que el tipo de objeto dado por asignar T a `string`: `{parents: [string, string], mate: string, children: string[]}`. Así que la siguiente asignación no generará errores:

```typescript
let aStringFamily: Family<string> = {
  parents: ['stern string', 'nice string'],
  mate: 'string next door', 
  children: ['stringy', 'stringo', 'stringina', 'stringolio']
};
```

En general, escribir tipos genéricos con la sintaxis `type typeName<T>` nos permite usar T dentro de la anotación de tipo como un **marcador de posición** para el tipo. Luego, cuando el tipo genérico es utilizado, T es reemplazado por el tipo proporcionado. (Escribir T es solo una convención. Podríamos usar S o `GenericType` igualmente).

¡Genial! Vamos a practicar con tipos genéricos.

---

## Generic Functions

También podemos usar los **genéricos** para crear colecciones de funciones tipadas. Las funciones genéricas como estas probablemente sean más fáciles de entender con un ejemplo. ¡Y por una vez, el ejemplo es realmente útil! Imagina que queremos crear una función que devuelva arrays llenos con un valor determinado. Vamos a escribir el código en JavaScript por ahora:

```javascript
function getFilledArray(value, n) {
  return Array(n).fill(value);
}
```

Aquí, `getFilledArray('cheese', 3)` da como resultado `['cheese', 'cheese', 'cheese']`. No hay problema, ¿verdad? Pues, encontramos un problema cuando intentamos especificar el tipo de retorno de la función. Sabemos que debería ser un array del tipo del valor proporcionado. ¿Tenemos que escribir una anotación de tipo separada para cada tipo de valor? ¡No! Aquí es donde entran las **funciones genéricas** para salvarnos.

```typescript
function getFilledArray<T>(value: T, n: number): T[] {
  return Array(n).fill(value);
}
```

El código anterior le dice a TypeScript que se asegure de que tanto `value` como el array devuelto tengan el mismo tipo `T`. Cuando se invoque la función, proporcionaremos el valor de `T`. Por ejemplo, podemos invocar la función usando `getFilledArray<string>('cheese', 3)`, lo que asigna a `T` el valor de `string`. Esto aún devuelve `['cheese', 'cheese', 'cheese']`, pero ahora la función está correctamente tipada y es menos propensa a errores. La función `getFilledArray<string>` es precisamente lo mismo que si hubiéramos escrito `(value: string, n: number): string[]` en su anotación de tipo.

En general, escribir funciones genéricas con la sintaxis `function functionName<T>` nos permite usar `T` dentro de la anotación de tipo como un **marcador de tipo**. Luego, cuando se invoque la función, `T` será reemplazado por el tipo proporcionado.

¡Increíble! Vamos a practicar con funciones genéricas.

-----

## Review

Al completar esta lección, ¡te has convertido oficialmente\* en un **héroe de TypeScript**! Ya no estás limitado a los tipos predefinidos de TypeScript; ¡ahora has aprendido a crear tus propios tipos personalizados! Estos incluyen:

* **Enums** (tanto de tipo cadena como numéricos)
* **Tipos de objetos**
* **Tipos de funciones**

Además, aprendiste a referenciar tipos complejos utilizando **alias de tipos**. ¡Y hasta lograste dominar los **genéricos**, que son como tipos personalizados doblemente! ¡Impresionante!

----