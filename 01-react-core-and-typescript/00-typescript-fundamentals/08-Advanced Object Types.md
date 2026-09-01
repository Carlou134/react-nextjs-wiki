# Advanced Object Types

## Introduction

Uno de los desafíos al escribir TypeScript es saber cómo aplicar los tipos en cada situación que encontraremos dentro de nuestro código. Echa un vistazo a este ejemplo:

```typescript
class Robot {
  identify(id: number) {
    console.log(`¡beep! Soy ${id}`);
  }
}
```

Aquí, hemos creado una clase llamada `Robot`. ¿Cómo podríamos aplicar el tipo `Robot`? Además, algunos robots pueden tener más funcionalidades que otros, o tener nombres de propiedades variables. ¿Cómo podríamos aplicar tipos en estas situaciones?

Esta lección trata sobre cómo podemos manejar una variedad de situaciones para asegurarnos de que nuestro código esté tipado, sin importar lo que haga nuestro programa o cómo esté estructurado. Los tipos siempre deberían ayudar a hacer nuestro código más seguro, sin imponer restricciones sobre cómo escribir y organizar nuestro código.

En esta lección, profundizaremos en cómo usar tipos con patrones de programación orientada a objetos, cómo usar tipos juntos para crear tipos combinados y mucho más.

-----

## Interfaces and Types

En TypeScript, podemos definir tipos de diversas maneras para ajustarlos a nuestro código. Hemos usado la palabra clave `type` para definir tipos, sin embargo, hay otra manera de definir tipos utilizando la palabra clave `interface`.

Aquí tienes un tipo que ya has visto antes:

```typescript
type Mail = {
  postagePrice: number;
  address: string;
}

const catalog: Mail = ...
```

Y aquí tienes un tipo idéntico que utiliza `interface`:

```typescript
interface Mail {
  postagePrice: number;
  address: string;
}

const catalog: Mail = ...
```

En este ejemplo, hemos usado tanto un `type` como una `interface` para crear un objeto tipado llamado `Mail`. Las sintaxis de `type` e `interface` son ligeramente diferentes, ya que `interface` no requiere un signo de igual (=) antes del objeto tipado. Funcionalmente, los dos tipos `Mail` en el ejemplo son idénticos: ambos impondrán el objeto tipado en tiempo de compilación cuando se tipeé una variable.

La mayor diferencia entre `interface` y `type` es que `interface` solo puede ser utilizada para tipar objetos, mientras que `type` puede ser utilizada para tipar objetos, **primitivos**, y más. Resulta que `type` es más versátil y funcional que `interface`. Entonces, ¿por qué usaríamos `interface`?

A veces, no queremos un tipo que pueda hacer todo. Nos gustaría que nuestros tipos estén más restringidos para que tengamos más probabilidades de escribir un código consistente. Dado que `interface` solo puede tipar objetos, es una opción perfecta para escribir programas orientados a objetos, porque estos programas necesitan muchos objetos tipados. Así que, comencemos a escribir tipos con `interface`.

----

## Interfaces and Classes

La palabra clave `interface` en TypeScript es especialmente útil para agregar tipos a una clase. Dado que `interface` está limitada a objetos tipados y usar clases es una manera de programar con objetos, `interface` y `class` hacen una excelente combinación.

TypeScript nos da la capacidad de aplicar un tipo a un objeto/clase con la palabra clave `implements`, de esta manera:

```typescript
interface Robot {
  identify: (id: number) => void;
}

class OneSeries implements Robot {
  identify(id: number) {
    console.log(`¡beep! Soy ${id.toFixed(2)}.`);
  }

  answerQuestion() {
    console.log('¡42!');
  }
}
```

En el ejemplo, hay una interfaz llamada `Robot` y una clase llamada `OneSeries`. Luego, se usa la palabra clave `implements` para aplicar el tipo `Robot` a `OneSeries`.

Observa que `Robot` tiene un método `.identify()`. Dado que `Robot` se aplica a `OneSeries`, `OneSeries` debe incluir un método llamado `.identify()` que coincida con la interfaz `Robot`. Además, `OneSeries` puede tener métodos y propiedades propias, como el método `.answerQuestion()`.

`implements` e `interface` nos permiten crear tipos que coincidan con una variedad de patrones de clases, lo que hace que `interface` sea una herramienta útil para usar en programas orientados a objetos.

-----

## Deep Types

A medida que nuestros programas crecen y se vuelven más complejos, necesitaremos agregar más métodos y propiedades a nuestros objetos para acomodar más funcionalidades. De hecho, puede que necesitemos agregar métodos y propiedades anidadas. Echa un vistazo a la siguiente clase:

```typescript
class OneSeries implements Robot {
  about;

  constructor(props: { general: { id: number; name: string; } }) {
    this.about = props;
  }

  getRobotId() {
    return `ID: ${this.about.general.id}`;
  }
}
```

En esta clase, `OneSeries` espera tener una propiedad `about` que es un objeto con un objeto anidado dentro de él. Dentro del método `getRobotId()`, `OneSeries` devuelve `this.about.general.id`. Para tipar un objeto anidado dentro de otro objeto, podríamos escribir una interfaz como esta:

```typescript
interface Robot {
  about: {
    general: {
      id: number;
      name: string;
    };
  };
  getRobotId: () => string;
}
```

Observa que dentro de la interfaz `Robot`, el objeto tipado `general` está anidado dentro del objeto tipado `about`. TypeScript nos permite anidar objetos de manera infinita para que podamos describir los datos correctamente.

----

## Composed Types

A medida que nuestros datos se anidan más profundamente, comenzaremos a tener objetos tipados que se vuelven difíciles de escribir y leer. Echa un vistazo al siguiente tipo:

```typescript
interface About {
  general: {
    id: number;
    name: string;
    version: {
      versionNumber: number;
    }
  }
}
```

Este tipo tiene dos niveles de anidamiento. Esto puede funcionar para un programa corto, pero a medida que nuestro código se expanda y necesitemos más características, probablemente nos enfrentaremos a dos problemas:

1. A medida que agreguemos más datos, esta interfaz puede volverse tan anidada que será difícil de leer tanto para nosotros como para otros desarrolladores.
2. Es probable que queramos solo una parte anidada de este tipo en algún lugar. Por ejemplo, puede que queramos solo el tipo del objeto `version` en nuestro programa, y sería útil poder usarlo sin todos los demás miembros de tipo en `About`.

Para resolver esto, TypeScript nos permite componer tipos. Podemos definir varios tipos y hacer referencia a ellos dentro de otros tipos. Aquí está el tipo de arriba, reescrito con tipos individuales compuestos:

```typescript
interface About {
  general: General;
}

interface General {
  id: number;
  name: string;
  version: Version;
}

interface Version {
  versionNumber: number;
}
```

El código resultante es un poco más largo, pero hemos resuelto los problemas de una gran interfaz: ahora podemos leer nuestro código más fácilmente con tipos nombrados y podemos reutilizar las interfaces más pequeñas en otros lugares de nuestro código.

Componer tipos es una forma esencial de mantener nuestro código organizado y flexible.

----

## Extending Interfaces

En TypeScript, no siempre es suficiente con poder componer tipos juntos. A veces, es conveniente copiar todos los miembros de tipo de un tipo a otro tipo. Podemos lograr esto con la palabra clave `extends`, como en este ejemplo:

```typescript
interface Shape {
  color: string;
}

interface Square extends Shape {
  sideLength: number;
}

const mySquare: Square = { sideLength: 10, color: 'blue' };
```

En este ejemplo, la interfaz `Square` usa la palabra clave `extends` para copiar todos los miembros de tipo de `Shape` en `Square`. Por lo tanto, cuando declaramos una variable como `mySquare`, se requieren tanto una propiedad `.sideLength` de tipo `number` como una propiedad `.color` de tipo `string`.

Usar `extends` puede ayudarnos a organizar nuestro código al abstraer miembros de tipo comunes en su propia interfaz, y luego copiarlos en tipos más específicos.

----

## Index Signatures

Cuando tipeamos objetos en TypeScript, a veces no es posible conocer los nombres de las propiedades de un objeto, como cuando obtenemos información de una fuente de datos externa o una API. Aunque no sepamos los nombres exactos de las propiedades en tiempo de compilación, es posible que sí sepamos cómo se verá la estructura de los datos en general. En ese caso, es útil escribir un tipo de objeto que nos permita incluir un nombre variable para el nombre de la propiedad. Esta característica se llama **firmas de índice** (index signatures).

Imagina que consultamos una API de mapas para obtener una lista de latitudes donde se puede ver un eclipse solar. Los datos podrían verse así:

```typescript
{
  '40.712776': true;
  '41.203323': true;
  '40.417286': false;
}
```

Sabemos que todos los nombres de las propiedades serán cadenas de texto (`string`), y que todos sus valores serán booleanos (`boolean`), pero no sabemos qué nombres tendrán esas propiedades. Para tipar este objeto, podemos utilizar una firma de índice para definir el tipo de este objeto. Podríamos escribir el tipo de este objeto así:

```typescript
interface SolarEclipse {
  [latitude: string]: boolean;
}
```

En el tipo `SolarEclipse`, se usa una firma de índice para definir un nombre de propiedad variable para cada miembro del tipo. La sintaxis `[latitude: string]` define que todos los nombres de las propiedades dentro de `SolarEclipse` son de tipo `string`, con un valor de tipo `boolean`. En la sintaxis `[latitude: string]`, el nombre `latitude` es solo para nosotros, los desarrolladores, como un nombre legible que aparecerá en mensajes de error potenciales más adelante.

----

## Optional Type Members

Un escenario común en la programación es crear una función o clase que pueda recibir muchos argumentos, algunos de los cuales son obligatorios y otros opcionales. Hasta ahora, cada interfaz en esta lección asume que todos los miembros de tipo son obligatorios, sin embargo, TypeScript nos permite hacer que algunos miembros de tipo sean opcionales. Echa un vistazo a este código:

```typescript
interface OptionsType {
  name: string;
  size?: string;
}

function listFile(options: OptionsType) {
  let fileName = options.name;

  if (options.size) {
    fileName = `${fileName}: ${options.size}`;
  }

  return fileName;
}
```

En el ejemplo, `OptionsType` tiene un miembro de tipo opcional llamado `size`. Podemos denotar cualquier miembro de tipo como opcional utilizando el operador `?` después del nombre de la propiedad y antes de los dos puntos (:), así: `size?: string;`. Dado que `.size` es opcional, agregamos una condición para verificar si existe antes de intentar usar la propiedad `.size`.

El parámetro opcional nos permite llamar a `listFile()` con un parámetro que no incluye la propiedad `size` en absoluto, así:

```typescript
listFile({ name: 'readme.txt' })
```

Los miembros de tipo opcionales en TypeScript nos permiten crear programas que no requieren que pasemos cada posible par clave-valor, lo que hace que nuestros programas sean más concisos y fáciles de entender.

---

## Review

¡🙌 Bien hecho! Has avanzado a través de toda esta lección. Nadie pondrá en duda tu superior conocimiento de TypeScript. Aquí tienes un resumen de lo que aprendimos:

* Podemos usar tanto las palabras clave `interface` como `type` para declarar tipos.
* `interface` es ideal para tipar objetos, especialmente dentro de programas orientados a objetos.
* Podemos aplicar una `interface` a una clase utilizando la palabra clave `implements`.
* Los tipos de objetos pueden anidarse infinitamente.
* Podemos definir múltiples tipos y componerlos para organizar nuestro código y hacerlo más flexible.
* Podemos copiar los miembros de tipo de una interfaz a otra utilizando la palabra clave `extends`.
* Podemos definir nombres de propiedades variables dentro de un tipo de objeto con una firma de índice. Una firma de índice usa una sintaxis como: `[propertyName: string]: string`.
* Es posible hacer que algunos miembros de tipo sean opcionales, utilizando el operador `?`. La sintaxis se ve como `name?: string`.

Usa el editor de código para perfeccionar tu comprensión sobre cómo tipar objetos en TypeScript.

----