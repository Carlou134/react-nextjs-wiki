# Union Types

## Introduction

TypeScript nos permite asignar tipos a las **variables** con diferentes niveles de especificidad. Si queremos asegurar que una variable sea una cadena de texto, podemos declararla como un `string`. Este tipo es muy específico, ya que TypeScript solo permitirá que la variable tenga un valor de tipo cadena.

En el otro extremo del espectro de especificidad, podríamos declarar una variable como `any`. Este tipo es muy poco específico. TypeScript permitirá que cualquier valor de cualquier tipo sea asignado sin generar quejas o errores.

Estos dos niveles de especificidad de tipos funcionan para muchas partes de nuestros programas. Sin embargo, a veces necesitamos encontrar un balance entre la especificidad extrema y ser totalmente imprecisos con nuestros tipos. Imagina que tenemos que escribir un programa que reciba la identificación de un empleado y luego imprima esa ID en la consola. El problema es que la ID de un empleado podría ser una cadena de texto o un número. Como necesitamos que nuestra variable ID permita más de un tipo, podríamos usar el tipo `any`, de esta forma:

```typescript
let ID: any;

console.log(`La ID es ${ID}.`);
```

El problema con el tipo `any` es que cualquier valor podría no funcionar bien con nuestro programa. Para solucionar esto, TypeScript nos permite ser flexibles con la especificidad de nuestros tipos al combinar diferentes tipos. Cuando combinamos tipos, se llama **unión**.

![union-diagram](/Images/union_diagram.svg)

---

## Defining Unions

Algunos valores pueden tener más de un tipo posible. TypeScript representa estos tipos "uno u otro" usando una **unión**.

Las uniones nos permiten definir múltiples tipos permitidos separando cada tipo con una barra vertical (`|`). Con una unión, podemos reescribir el programa del ejercicio anterior así:

```typescript
let ID: string | number;

// número
ID = 1;

// o cadena
ID = '001';

console.log(`La ID es ${ID}.`);
```

En este ejemplo, `string | number` es una **unión** que permite que `ID` sea una cadena (`string`) o un número (`number`). Es más flexible que usar un solo tipo primitivo, pero mucho más específico que el tipo `any`.

Las uniones se pueden escribir en cualquier lugar donde se defina un tipo, incluyendo los parámetros de funciones:

```typescript
function getMarginLeft(margin: string | number) {
  return { 'marginLeft': margin };
}
```

Usar uniones para tipar los parámetros de funciones es especialmente útil porque las funciones suelen necesitar manejar múltiples tipos de entrada.

---

## Type Narrowing

El uso de **uniones** nos da más flexibilidad con la especificidad de los tipos, pero también hay más aspectos a considerar. Por ejemplo, observa esta unión:

```typescript
function getMarginLeft(margin: string | number) {
  // ...
}
```

Dado que `margin` puede ser una cadena (`string`) o un número (`number`), podríamos querer ejecutar diferentes lógicas en el cuerpo de la función `getMarginLeft()`, dependiendo de si es una cadena o un número. Para hacerlo, podemos implementar un **type guard** (guardián de tipo).
Un *type guard* es una condición que verifica si una variable es de un tipo específico, como en este ejemplo:

```typescript
function getMarginLeft(margin: string | number) {
  // margin puede ser una cadena o un número aquí
  
  if (typeof margin === 'string') {
    // aquí margin debe ser una cadena
  }
}
```

En el ejemplo anterior, TypeScript es capaz de leer el *type guard* e inferir que la variable `margin` dentro de esa condición debe ser una cadena.
Dado que TypeScript sabe que `margin` es una cadena, nos permitirá usar métodos específicos de las cadenas, como este:

```typescript
if (typeof margin === 'string') {
  return margin.toLowerCase();
}
```

Si intentamos llamar a `margin.toLowerCase()` fuera del *type guard* que verifica que es una cadena, TypeScript generaría un error, indicando que el método `.toLowerCase()` no existe en los valores de tipo número.
Este error ocurre porque `margin` está tipado como una unión `string | number`.

Este concepto se llama **type narrowing** (reducción de tipo).
La **reducción de tipo** es un proceso en TypeScript que refina un valor con múltiples tipos en un solo tipo específico.
En nuestros ejemplos, TypeScript ha reducido el tipo dentro del *type guard* a solo una cadena.
La reducción de tipo nos permite usar uniones y luego aplicar lógica específica para cada tipo, sin que TypeScript interfiera.

---

## Inferred Union Return Types

Una de las cosas increíbles de TypeScript es que puede inferir los tipos en muchos casos, por lo que no tenemos que escribirlos manualmente. Un gran ejemplo de esto es el tipo de retorno de una función. TypeScript examina el contenido de una función e infiere qué tipos puede devolver. Si hay varios tipos de retorno posibles, TypeScript inferirá el tipo de retorno como una **unión**.

Por ejemplo, toma este caso, donde llamamos a una función llamada `getBookFromServer()`, que podría fallar:

```typescript
function getBook() {
  try {
    return getBookFromServer();
  } catch (error) {
    return `Algo salió mal: ${error}`;
  }
}
```

Si la llamada es exitosa, la función devolverá un tipo `Book` que describe un libro. Si la llamada falla, la función devolverá una cadena (`string`). `getBook()` puede devolver un tipo `Book` o `string`, y TypeScript infiere el tipo de retorno como la unión `Book | string`.
Como TypeScript puede inferir el tipo de retorno de la función, no necesitamos definirlo manualmente.

---

## Unions and Arrays

Las **uniones** son aún más poderosas cuando se utilizan en combinación con **arreglos**.

Por ejemplo, podemos representar el tiempo en TypeScript con un tipo `number` o `string`. Si tuviéramos una lista de fechas en ambos tipos, necesitaríamos un arreglo que permita valores de tipo `string` y `number`. Las uniones vienen a ayudarnos con esto.

Para crear una unión que soporte múltiples tipos para los valores de un arreglo, debemos envolver la unión entre paréntesis (`(string | number)`), luego usar la notación de arreglo (`[]`).

```typescript
const dateNumber = new Date().getTime(); // devuelve un número
const dateString = new Date().toString(); // devuelve una cadena

const timesList: (string | number)[] = [dateNumber, dateString];
```

En el ejemplo anterior, la variable `timesList` está tipada para permitir los tipos `string` y `number` como valores dentro de su arreglo. Si tratamos de agregar un valor a `timesList` que no sea de esos tipos, como en `timesList.push(true)`, TypeScript mostraría un error indicando que los tipos `boolean` no están permitidos dentro de `timesList`.

Una última cosa: los **paréntesis** son cruciales para tipar correctamente los arreglos. Si omitiéramos los paréntesis y escribiéramos `string | number[]`, ese tipo permitiría cadenas o arreglos que contengan **solo números**.

---

## Common Key Value Pairs

Cuando ponemos miembros de tipo en una **unión**, TypeScript solo nos permitirá usar los métodos y propiedades comunes que todos los miembros de la unión comparten. Mira este código:

```typescript
const batteryStatus: boolean | number = false;

batteryStatus.toString(); // No hay error de TypeScript
batteryStatus.toFixed(2); // Error de TypeScript
```

Dado que `batteryStatus` puede ser un `boolean` o un `number`, TypeScript solo nos permitirá llamar a los métodos que tanto `number` como `boolean` comparten. Ambos comparten el método `.toString()`, así que no hay problema allí. Pero, dado que solo `number` tiene el método `.toFixed()`, TypeScript marcará un error si intentamos llamarlo.

Esta regla también se aplica a los **objetos de tipo** que definimos. Mira este código:

```typescript
type Goose = { 
  isPettable: boolean; 
  hasFeathers: boolean;
  canThwartAPicnic: boolean;
}

type Moose = {
  isPettable: boolean; 
  hasHoofs: boolean;
}

const pettingZooAnimal: Goose | Moose = { isPettable: true };

console.log(pettingZooAnimal.isPettable); // No hay error de TypeScript
console.log(pettingZooAnimal.hasHoofs); // Error de TypeScript
```

Como antes, dado que `.isPettable` está presente en los tipos `Goose` y `Moose`, TypeScript nos permite llamarlo. Pero, como `.hasHoofs` solo es una propiedad de `Moose`, no podemos acceder a ella desde `pettingZooAnimal`. Cualquier propiedad o método que no sea compartido por todos los miembros de la unión no será permitido y producirá un error de TypeScript.

---

## Unions with Literal Types

Podemos usar **tipos literales** con las **uniones de TypeScript**. Las uniones de tipos literales son útiles cuando queremos crear estados distintos dentro de un programa.

Por ejemplo, si estuviéramos escribiendo el código que controla los semáforos, podríamos escribir un programa como este:

```typescript
type Color = 'green' | 'yellow' | 'red';

function changeLight(color: Color) {
  // ...
}
```

Con el código anterior, podríamos asegurarnos de que, cada vez que se llame a `changeLight()`, se pase solo uno de los colores permitidos para el semáforo. Si intentáramos llamar a `changeLight('purple')`, TypeScript generaría un error, ya que ese no es un color válido para un semáforo.

Esta técnica nos permite escribir **funciones** que son específicas sobre los estados que pueden manejar, lo que nos ayuda a escribir código menos propenso a errores.

---

## Review Unions

🙌 ¡Bien hecho! Hemos aprendido diversas formas de crear tipos tan específicos como necesitemos con **uniones**. Para resumir, hemos aprendido:

* Podemos combinar múltiples tipos con el carácter de barra vertical (`|`). Esta es la sintaxis para definir una unión. Cada tipo en una unión se llama **miembro de tipo**.
* Podemos **reducir** qué métodos y propiedades están disponibles en un programa mediante la **reducción de tipo**. La **reducción de tipo** nos permite tipar una variable como una unión y luego reducir la unión con un **type guard** para llamar a métodos y propiedades específicas de cada miembro de la unión.
* Si una función puede devolver múltiples tipos, TypeScript inferirá todos los tipos de retorno posibles como una unión.
* Podemos usar uniones para permitir que los **arreglos** tengan valores de múltiples tipos.
* Para llamar a un método o propiedad en una variable tipada como una unión, solo podemos llamar a métodos o propiedades que sean idénticos en todos los miembros de la unión.
* Podemos definir estados dentro de nuestro programa utilizando **tipos literales** y **uniones**.

¡Espero que te haya quedado claro! Si necesitas más detalles o tienes preguntas, ¡aquí estoy para ayudarte!

----