# Arrays

## Introduction

**Nuestro viaje con TypeScript nos ha llevado ahora a un nuevo destino: Array-bia. El tipado de **arreglos** en TypeScript es un poco diferente que trabajar con tipos primitivos. Esto se debe a que los arreglos suelen contener muchos elementos de datos. Mantener un seguimiento del tipo del arreglo significa mantener un seguimiento del tipo de cada elemento. Por ejemplo:**

```ts
let firstArray = [1, 2, 3, 4];
let secondArray = [5, '6', [7]];
```

Podemos ver que `firstArray` tiene elementos que son todos del tipo `number`. Por otro lado, `secondArray` tiene elementos de tipos variados: `number`, `string` y `Array`. Ambos son ejemplos de arreglos válidos en JavaScript. Como pronto exploraremos, TypeScript facilita mucho el seguimiento de los tipos de elementos en arreglos como los anteriores.

**Por ahora, sin embargo, hagamos la vida más difícil haciendo de cuenta que TypeScript no tiene maneras especiales de tipar arreglos. ¿Qué tan difícil sería hacer cumplir la consistencia de tipos?**

---

## Array Type Annotations

**La anotación de tipo de TypeScript para los tipos de arreglos es bastante simple: ponemos `[]` después del tipo de elemento. En este código, `names` es un arreglo que solo puede contener cadenas (`strings`):**

```ts
let names: string[] = ['Danny', 'Samantha'];
```

**Un método alternativo es usar la sintaxis `Array<T>`, donde `T` representa el tipo.**

```ts
let names: Array<string> = ['Danny', 'Samantha'];
```

**En el código anterior, el tipo `T` es `string`. No usaremos la sintaxis `Array<T>` en esta lección, pero es bueno familiarizarse con ella.**

Como podríamos esperar, obtenemos un error de tipo si intentamos asignar un arreglo de números a una variable `string[]`:

```ts
let names: string[] = [1, 2, 3]; // ¡Error de tipo!
```

Eso fue como un error de asignación con tipos primitivos. Sin embargo, los arreglos de TypeScript también pueden lanzar errores cuando se agregan elementos del tipo incorrecto:

```ts
let names: string[] = ['Damien'];
names.push(666); // ¡Error de tipo!
```

**¡Vamos a practicar algunas anotaciones de tipo para arreglos!**

---

## Multi-dimensional Arrays

**Hasta ahora hemos visto arreglos de tipo `string[]`, pero también podríamos tener arreglos que solo contengan números (`number[]`) o booleanos (`boolean[]`). De hecho, podemos crear arreglos de cualquier tipo. También podemos declarar arreglos multidimensionales: arreglos de arreglos (de algún tipo).**

```ts
let arr: string[][] = [['str1', 'str2'], ['more', 'strings']];
```

**Piensa en `string[][]` arriba como una forma corta de `(string[])[]`, es decir, un arreglo donde cada elemento es un `string[]`. Exploraremos tipos de arreglos más complejos en lecciones posteriores.**

**El arreglo vacío (`[]`) es compatible con cualquier tipo de arreglo:**

```ts
let names: string[] = []; // Sin errores de tipo.
let numbers: number[] = []; // Sin errores de tipo.
names.push('Isabella');  
numbers.push(30);
```

**¡Es hora de practicar!**

---

## Tuples

**Hasta ahora hemos visto arreglos con todos los elementos del mismo tipo. Pero, como sabemos, los arreglos en JavaScript son flexibles y pueden tener elementos de tipos diferentes. Con TypeScript, también podemos definir arreglos con una secuencia fija de tipos:**

```ts
let ourTuple: [string, number, string, boolean] = ['Is', 7 , 'our favorite number?' , false];
```

**En TypeScript, cuando un arreglo está tipado con elementos de tipos específicos, se llama tupla. La tupla anterior (`ourTuple`) contiene los elementos: 'Is', 7, 'our favorite number?', false y la tupla tiene un tipo de `[string, number, string, boolean]`. Los tipos de tupla especifican tanto las longitudes como los órdenes de las tuplas compatibles, y causarán un error si no se cumplen estas condiciones:**

```ts
let numbersTuple: [number, number, number] = [1,2,3,4]; // ¡Error de tipo! numbersTuple solo debe tener tres elementos.
let mixedTuple: [number, string, boolean] = ['hi', 3, true]; // ¡Error de tipo! El primer elemento debería ser un número, el segundo una cadena y el tercero un booleano.
```

**En cuanto a JavaScript, las tuplas actúan de la misma manera que los arreglos. Ambos tienen la propiedad `.length`. Podemos acceder (o cambiar) los elementos de ambos usando `[índice]`. Pero a pesar de sus similitudes, las tuplas y los arreglos no tienen tipos compatibles dentro de TypeScript. Específicamente, no podemos asignar un arreglo a una variable de tipo tupla, incluso cuando los elementos son del tipo correcto:**

```ts
let tup: [string, string] = ['hi', 'bye'];
let arr: string[] = ['there', 'there'];
tup = ['there', 'there']; // Sin errores.
tup = arr; // ¡Error de tipo! No se puede asignar un arreglo a una tupla.
```

**¡Ahora practiquemos usando tuplas y tipos de tuplas!**

---

## Array Type Inference

**TypeScript puede inferir los tipos de variables a partir de los valores iniciales y las declaraciones de retorno. Aun así, puede que no sepamos exactamente qué tipo de inferencia esperar cuando tratamos con arreglos. Por ejemplo:**

```ts
let examAnswers = [true, false, false];
```

**¿Cuál es el tipo de `examAnswers`? Podría ser igualmente un `boolean[]` o una tupla `[boolean, boolean, boolean]`. En realidad, siempre es el primero de estos, ya que es el tipo menos restrictivo. Esto nos permite expandir el arreglo:**

```ts
examAnswers[3] = true; // Sin error de tipo.
```

**Dado que las tuplas tienen longitudes fijas, no podríamos agregar elementos booleanos adicionales a una tupla:**

```ts
let tupleOfExamAnswers: [boolean, boolean, boolean] = [true, false, false];
tupleOfExamAnswers[3] = true; // ¡Error de tipo! La tupla solo tiene 3 elementos.
```

**También obtenemos el mismo tipo de inferencia cuando usamos el método `.concat()`:**

```ts
let tup: [number, number, number] = [1, 2, 3];
let concatResult = tup.concat([4, 5, 6]); // concatResult tiene el valor [1, 2, 3, 4, 5, 6].
```

**En el código anterior, TypeScript infiere que la variable `concatResult` es un arreglo de números, no una tupla.**

**La lección aquí es que la inferencia de tipos devuelve arreglos. Cuando queremos tuplas, necesitamos usar anotaciones de tipo explícitas.**

---

## Rest Parameters

**Asignar tipos a parámetros rest es similar a asignar tipos a arreglos. Aquí tienes un ejemplo de parámetro rest sin tipos:**

```ts
function smush(firstString, ...otherStrings) {
  let output = firstString;
  for (let i = 0; i < otherStrings.length; i++) {
    output = output.concat(otherStrings[i]);
  }
  return output;
}
```

**Esta función concatena todos sus argumentos. Por ejemplo, al llamar a `smush('hi ', 'there')`, se devuelve el valor 'hi there'.** El parámetro rest `otherStrings` permite que la función trabaje con cualquier cantidad de parámetros mayor a cero:

```ts
smush('a', 'h', 'h', 'H', 'H', 'H', '!', '!'); // Devuelve: 'ahhHHH!!'.
```

**La función funciona bien, pero no es segura en cuanto a tipos. No queremos que un programador cometa un error como `smush(1, 2, 3)`, ya que eso causaría un error. ¡TypeScript al rescate! Las anotaciones de tipo para un parámetro rest son idénticas a las anotaciones de tipo para arreglos. La función con un parámetro rest correctamente tipado sería:**

```ts
function smush(firstString, ...otherStrings: string[]) {
  /*resto de la función*/
}
```

**Con este cambio, TypeScript tratará `otherStrings` como un arreglo de cadenas. Esto significa que `smush(1, 2, 3)` causará un error de tipo porque \[2, 3] no es un arreglo de cadenas.**

**Ahora, ¡es tu turno de escribir un tipo para un parámetro rest!**

---

## Spread Syntax

**Como los mejores vinos y quesos, las tuplas de TypeScript combinan perfectamente con la sintaxis spread de JavaScript. Esto es muy útil para llamadas a funciones que usan muchos argumentos, como esta:**

```ts
function gpsNavigate(
  startLatitudeDegrees: number, startLatitudeMinutes: number, startNorthOrSouth: string,
  startLongitudeDegrees: number, startLongitudeMinutes: number, startEastOrWest: string,
  endLatitudeDegrees: number, endLatitudeMinutes: number, endNorthOrSouth: string,
  endLongitudeDegrees: number, endLongitudeMinutes: number, endEastOrWest: string
) {
  /* subrutina de navegación aquí */
}
```

**La llamada a la función `gpsNavigate(40, 43.2, 'N', 73, 59.8, 'W', 25, 0, 'N', 71, 0, 'W')` calcula una ruta desde las oficinas de Codecademy en la ciudad de Nueva York (40 grados 43.2 minutos al norte, 73 grados 59.8 minutos al oeste) hasta coordenadas seleccionadas en el Triángulo de las Bermudas. Todos estamos de acuerdo en que esta llamada a la función es difícil de leer.**

**En su lugar, podemos usar variables de tuplas que representen las coordenadas de inicio y final:**

```ts
let codecademyCoordinates: [number, number, string, number, number, string] = [40, 43.2, 'N', 73, 59.8, 'W'];
let bermudaTCoordinates: [number, number, string, number, number, string] = [25, 0, 'N', 71, 0, 'W'];
```

**Estas anotaciones de tipo de tupla garantizan que los tipos de los elementos sean parámetros válidos para la función `gpsNavigate()`.**

**Ahora, usamos la sintaxis spread de JavaScript para escribir una llamada a la función mucho más legible:**

```ts
gpsNavigate(...codecademyCoordinates, ...bermudaTCoordinates);
```

**Y, por cierto, esto también hace que el viaje de regreso sea realmente conveniente de calcular:**

```ts
gpsNavigate(...bermudaTCoordinates, ...codecademyCoordinates);
// Si hay un viaje de regreso...
```

**Ahora, ¡es tu turno de practicar!**

---

## Review

¡Precioso! Los tipos de arrays pueden parecer un tema monstruoso, pero ahora los tienes bajo control. Aprendiste:

* Las **anotaciones de tipo** para **arrays**.
* Qué son las **tuplas** y cómo hacer sus anotaciones de tipo.
* Cómo funciona la inferencia de tipos con arrays y tuplas.
* Cómo usar los operadores **rest** y **spread** con TypeScript.

Los tipos de arrays son una parte crucial de TypeScript, ¡así que excelente trabajo!

----