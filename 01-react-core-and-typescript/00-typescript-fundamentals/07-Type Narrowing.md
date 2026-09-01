# Type Narrowing

## Introduction

Al compilar código TypeScript a JavaScript, el compilador generará errores relacionados con los tipos de variables. Este proceso de compilación implica darle al compilador de TypeScript la información que necesita para realizar verificaciones de tipos. Por lo tanto, cuando asignamos a nuestras **variables** tipos específicos, estos tipos son reforzados por el compilador/TypeScript.

Si TypeScript solo pudiera verificar los tipos en el momento de la compilación, eso todavía sería útil, pero resulta que TypeScript puede hacer mucho más. TypeScript tiene la capacidad de evaluar cómo se comportará nuestro código en tiempo de ejecución y luego deducir los tipos de las variables por nosotros. Cuando el compilador detecta un error, se quejará al ejecutar `tsc` y también mostrará una línea roja ondulada debajo del código en el editor.

Evaluar el código en tiempo de ejecución puede ser realmente útil cuando nuestros programas tienen variables de múltiples tipos. En el caso de **uniones**, TypeScript puede enfocarse en un tipo más específico evaluando nuestro código en tiempo de ejecución. Por ejemplo:

```typescript
function formatDate(date: string | number) {
  // date puede ser un número o una cadena aquí

  if (typeof date === 'string') {
    // date debe ser una cadena aquí
  }
}
```

En el ejemplo anterior, el parámetro `date` tiene un tipo de unión `string | number`. Dentro del cuerpo de `formatDate()`, hay una condición que verifica si `date` es una 'cadena'. Si se cumple la condición, TypeScript puede inferir que `date` debe ser de tipo `string` dentro de esa condición. Esto nos permitirá llamar métodos y propiedades que son válidos para los tipos `string`. Esto se llama **narrowing** de tipo. **Narrowing** de tipo es cuando TypeScript puede deducir tipos más específicos basándose en el código circundante de la variable.

---

## Type guards

Una forma en que TypeScript puede reducir un tipo es mediante una declaración condicional que verifica si una variable es de un tipo específico. Este patrón se llama **guarda de tipo** (type guard). Las guardas de tipo pueden usar una variedad de operadores que verifican el tipo de una variable. Uno de los operadores que podemos usar es `typeof`. Por ejemplo:

```typescript
function formatDate(date: string | number) {
  // date puede ser un número o una cadena aquí

  if (typeof date === 'string') { 
    // date debe ser de tipo string
  }
}
```

En este ejemplo, TypeScript es capaz de deducir que `date` debe ser un tipo 'string' dentro de la condición porque `typeof` verificó que `date` es un 'string'. TypeScript evaluó lo que haría nuestro código en tiempo de ejecución y luego dedujo un tipo más específico para `date`.

TypeScript puede reconocer las guardas de tipo `typeof` que verifican estos valores específicos: 'string', 'number', 'boolean' y 'symbol'.

---

## Using in with Type Guards

A medida que escribimos más tipos, es inevitable que creemos tipos personalizados para describir mejor las propiedades y métodos de nuestros datos. Aunque usar `typeof` puede llevarnos bastante lejos, a veces queremos verificar si un método específico existe en un tipo, en lugar de un tipo como 'string'. Ahí es donde entra en juego el operador `in`. El operador `in` verifica si una propiedad existe en un objeto o en cualquier parte de su cadena de prototipos. Echa un vistazo a este ejemplo:

```typescript
type Tennis = {
  serve: () => void;
}

type Soccer = {
  kick: () => void;
}

function play(sport: Tennis | Soccer) {
  if ('serve' in sport) {
    return sport.serve();
  }

  if ('kick' in sport) {
    return sport.kick();
  }
}
```

En el ejemplo de arriba, verificamos si la propiedad 'serve' existe en `sport` con el operador `in`. La propiedad 'serve' debe existir dentro de la condición, por lo que TypeScript puede reducir el tipo de `sport` dentro de la condición a tipo `Tennis`. Este **narrowing** de tipo es posible porque TypeScript reconoce `in` como una guarda de tipo.

---

## Narrowing with else

En nuestros ejemplos anteriores, hemos utilizado dos declaraciones `if` separadas como guardas de tipo para manejar cada tipo posible. Resulta que TypeScript puede reconocer el bloque `else` de una declaración `if/else` como el chequeo de guarda de tipo opuesto al chequeo de guarda de tipo de la declaración `if`. Por ejemplo:

```typescript
function formatPadding(padding: string | number) {
  if (typeof padding === 'string') {
    return padding.toLowerCase();
  } else {
    return `${padding}px`;
  }
}
```

En el ejemplo de arriba, la función `formatPadding()` puede recibir un valor para el padding de CSS, como el número 32 o la cadena `'0 32px'`. Luego, se usa una guarda de tipo para determinar cómo formatear el argumento de padding.

La guarda de tipo `typeof padding === 'string'` le dice a TypeScript que dentro del bloque `if`, `padding` debe ser de tipo `string`. Más allá de eso, TypeScript también puede deducir que, dado que el argumento `padding` está tipado como la unión `string | number`, dentro del bloque `else`, `padding` debe ser de tipo `number`.

Dado que TypeScript puede entender cómo funcionará nuestro código en tiempo de ejecución, es capaz de deducir tipos específicos para nosotros, como sucede con el `else` de una declaración `if/else`.

---

## Narrowing After a Type Guard

La capacidad de TypeScript para inferir tipos después de una guarda de tipo va aún más allá de inferir el tipo dentro de una declaración `else`. TypeScript también puede hacer un **narrowing** de tipo sin necesidad de una declaración `else`, siempre que haya una declaración `return` dentro de la guarda de tipo. Echa un vistazo a este ejemplo:

```typescript
type Tea = {
  steep: () => string;
}

type Coffee = {
  pourOver: () => string;
} 

function brew(beverage: Coffee | Tea) {
  if ('steep' in beverage) {
    return beverage.steep();
  }

  return beverage.pourOver();
}
```

En la condición, inmediatamente retornamos `beverage.steep()`. Una vez que encontramos una declaración `return`, la ejecución de la función se detiene. Cualquier código que estaba destinado a trabajar con una bebida de tipo `Tea` se ejecutará y se devolverá dentro de la condición. Luego, TypeScript infiere que el código posterior a la condición debe ser de tipo `Coffee`.

---

## Review Type Narrowing

¡Eres el tipo de persona que termina las lecciones! ¡Buen trabajo completando **Type Narrowing**! Ahora estás listo para aprovechar los poderes de inferencia de TypeScript mediante las guardas de tipo y el narrowing de tipos. Vamos a repasar lo que aprendimos:

* TypeScript puede entender cómo se ejecutará nuestro código en tiempo de ejecución, de modo que puede inferir tipos más específicos mientras escribimos código. Esto se llama **type narrowing**.
* Una expresión que verifica si una variable es de un tipo específico se llama **type guard**. Las **type guards** permiten que TypeScript reconozca cuándo puede hacer un narrowing de tipo.
* El operador `typeof` es útil cuando escribimos **type guards**. Puede verificar si una variable es de tipo 'string', 'number', 'boolean' o 'symbol'.
* El operador `in` es útil para verificar si una propiedad específica existe en un objeto. `in` es especialmente útil cuando tenemos datos representados como objetos.
* TypeScript puede hacer un narrowing de tipo después de una **type guard** con un bloque `else`. TypeScript entiende que el bloque `else` de una declaración `if` debe ser la condición inversa de la condición del `if`.
* TypeScript puede ir aún más allá y hacer un narrowing de tipo después de una **type guard** si la guarda de tipo tiene una declaración `return` u otra declaración terminal dentro de su bloque, sin necesidad de un `else`.

---

