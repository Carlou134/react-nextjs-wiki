# Types

## From JavaScript to TypeScript

**Inventado en 1995, JavaScript fue diseñado como un pequeño lenguaje de scripting para páginas web simples en navegadores. No fue hasta 1999 que JavaScript tuvo la capacidad de soportar el tipo de páginas web dinámicas que vemos hoy en día, y no fue una práctica común usar JavaScript de esa manera hasta 2005.**

Para cumplir con su caso de uso original, JavaScript fue diseñado para ser muy flexible y fácil de usar en aplicaciones pequeñas. Estas características hacen que JavaScript sea un excelente primer lenguaje para aprender, pero también lo hacen menos ideal para construir aplicaciones a gran escala con cientos o incluso miles de archivos. Lenguajes de programación más estrictos informan al desarrollador cuando cambia una parte del código de una forma que puede romper otras áreas. JavaScript no lo hace, lo que a menudo conduce a comportamientos inesperados en tiempo de ejecución.

Para abordar estas deficiencias, Microsoft desarrolló TypeScript y lo lanzó públicamente en 2012, con el objetivo de combinar la flexibilidad de JavaScript con las ventajas de un lenguaje más estricto.

**TypeScript es un lenguaje de programación que añade tipos a JavaScript.** Nos permite escribir JavaScript con un conjunto de herramientas llamado *sistema de tipos*, que puede detectar errores potenciales, aclarar la estructura del código y ayudar a refactorizarlo. Además, TypeScript incorporó características más modernas del lenguaje JavaScript, como las funciones flecha y las clases, años antes de que se añadieran oficialmente a JavaScript.

Hoy en día, TypeScript es uno de los lenguajes más apreciados del mundo, desde proyectos de código abierto como Angular y Webpack, hasta desarrollos a gran escala en empresas como Amazon y Google. ¡Nosotros lo usamos en Codecademy!

---

## What is TypeScript?

**Ahora que ya sabemos por qué existe TypeScript, hablemos de cómo lo usamos:**

1. Primero, escribimos código TypeScript en archivos con la extensión **.ts**.
2. Luego, ejecutamos nuestro código a través del **transpilador de TypeScript**. El transpilador comprobará que el código cumpla con los estándares de TypeScript y mostrará errores si no lo hace.
3. Si el código TypeScript puede convertirse en JavaScript funcional, el transpilador generará una versión JavaScript del archivo (**.js**).
4. El código TypeScript es un **superset** (superconjunto) del código JavaScript: tiene todas las características del JavaScript tradicional, pero añade algunas nuevas.

Dado este código TypeScript como entrada:

```ts
let firstName = 'Anders';
```

El transpilador de TypeScript generaría el siguiente código JavaScript:

```js
let firstName = 'Anders';
```

¡Así es! ¡Son iguales! El código TypeScript, en general, se ve muy similar al código JavaScript equivalente.

**Lo que aprenderemos a lo largo de esta lección y del curso es la verdadera ventaja de usar TypeScript: que nos ayuda a entender mejor nuestro código y, en particular, a descubrir errores.**

### 1. Ejecutar un archivo `.ts` (TypeScript)

👉 Primero necesitas **Node.js** y **TypeScript** instalados.

* Instala TypeScript globalmente:

```bash
npm install -g typescript
```

* Para ejecutar directamente un `.ts` sin compilar manualmente, instala **ts-node**:

```bash
npm install -g ts-node
```

* Ejecuta tu archivo:

```bash
ts-node archivo.ts
```

### 2. Convertir `.ts` a `.js` (compilar)

👉 Con TypeScript ya instalado, compilas así:

```bash
tsc archivo.ts
```

Esto genera un archivo **`archivo.js`** que puedes correr con Node:

```bash
node archivo.js
```

🔑 En resumen:

* **`ts-node archivo.ts`** → ejecuta directo.
* **`tsc archivo.ts` + `node archivo.js`** → lo convierte primero a JS y luego lo corres.

----

## Type Inferences

**JavaScript nos permite asignar cualquier valor a cualquier variable.** Esto lo hace muy flexible de usar, lo cual puede ser útil para comenzar rápidamente a programar. En la práctica, sin embargo, las variables a las que se les asignan valores de distintos tipos a lo largo de un programa pueden ser confusas o provocar errores.

Una de las primeras cosas que descubriremos con TypeScript es que, cuando declaramos una variable con un valor inicial, **esa variable no puede ser reasignada con un valor de un tipo de dato diferente**. Esto es un ejemplo de *inferencia de tipos*: en todo el programa, TypeScript espera que el tipo de dato de la variable coincida con el tipo del valor que se le asignó inicialmente al momento de declararla.

TypeScript reconoce los tipos de datos *primitivos* incorporados en JavaScript:

* **boolean**
* **number**
* **null**
* **string**
* **undefined**

Si intentamos reasignar una variable con un valor de otro tipo, TypeScript generará un error.

```ts
let order = 'first';

order = 1;
```

Al ejecutar el código TypeScript anterior, se mostrará el siguiente error en la terminal:
**Type 'number' is not assignable to type 'string'.**

El sistema de tipos de TypeScript nos está diciendo que `order` se espera que sea del tipo primitivo `string`, pero estamos intentando asignarle un valor de tipo `number`. Eso no está permitido: **las variables solo pueden recibir valores del tipo que espera el sistema de tipos.**

Podemos corregir este error cambiando el nuevo valor al tipo `string` esperado:

```ts
let order = 'first';

order = '1';
```

----

## Type Shapes

**Debido a que TypeScript sabe qué tipos tienen nuestros objetos, también sabe qué formas siguen esos objetos.** La forma de un objeto describe, entre otras cosas, qué propiedades y métodos contiene o no contiene.

Los tipos incorporados en JavaScript tienen propiedades y métodos conocidos que siempre existen. Por ejemplo, todas las cadenas de texto tienen una propiedad `.length` y un método `.toLowerCase()`.

```js
"OH".length; // 2
"MY".toLowerCase(); // "my"
```

El comando `tsc` de TypeScript te avisará si tu código intenta acceder a propiedades y métodos que no existen:

```ts
"MY".toLowercase();
// Property 'toLowercase' does not exist on type '"MY"'.
// Did you mean 'toLowerCase'?
```

A través de este conocimiento sobre las formas de los tipos, **TypeScript nos ayuda a localizar rápidamente errores en nuestro código.**

¡Vamos a intentarlo!

----

## Any

**Hay algunos casos en los que TypeScript no intentará inferir qué tipo tiene algo,** generalmente cuando una variable es declarada sin asignarle un valor inicial. En situaciones donde no puede inferir un tipo, TypeScript considerará que la variable es de tipo `any`.

Las variables de tipo `any` pueden ser asignadas a cualquier valor, y TypeScript no generará un error si luego se les reasigna un valor de un tipo diferente.

```ts
let onOrOff;

onOrOff = 1;
onOrOff = false;
```

En el código anterior, declaramos la variable `onOrOff` sin un valor inicial. TypeScript la considera de tipo `any`, por lo tanto, no produce un error cuando cambiamos la asignación de la variable de un valor numérico a un valor booleano.

----

## Variable Type Annotations

**En algunas situaciones, nos gustaría declarar una variable sin un valor inicial, pero asegurarnos de que solo se le asignen valores de un tipo específico.** Si la dejamos como `any`, TypeScript no podrá protegernos de asignarle accidentalmente un valor de un tipo incorrecto que podría romper nuestro código.

Podemos indicarle a TypeScript qué tipo tiene algo o qué tipo tendrá utilizando una *anotación de tipo*.

Las variables pueden tener anotaciones de tipo (también conocidas como declaraciones de tipo) agregadas justo después de sus nombres. Proporcionamos una anotación de tipo añadiendo dos puntos (`:`) seguidos del tipo (por ejemplo, `number`, `string` o `any`).

```ts
let mustBeAString : string;
mustBeAString = 'Catdog';

mustBeAString = 1337;
// Error: Type 'number' is not assignable to type 'string'
```

En el código anterior, declaramos explícitamente que `mustBeAString` debe ser de tipo `string` sin asignarle un valor inicial. Esto nos permite asignarle el valor `'Catdog'` sin problemas, pero cuando intentamos asignarle un valor numérico más adelante, TypeScript nos muestra un mensaje de error indicándonos que no se puede asignar un número a una variable de tipo `string`.

¡Ya estamos familiarizándonos con lo útil que pueden ser las *anotaciones de tipo*! Algunos desarrolladores pueden encontrar que las anotaciones de tipo hacen que el código sea más verboso o difícil de entender para otros, sin embargo, **se eliminan automáticamente cuando se compila a JavaScript.**

---

## Review

¡Genial! Has dado tus primeros pasos en el maravilloso mundo de la seguridad de tipos con TypeScript. 💪

Para recapitular, has aprendido:

* TypeScript es un superset de JavaScript que añade tipos.
* El sistema de tipos se refiere a la comprensión de TypeScript sobre cómo debe funcionar tu código: principalmente, qué tipos de datos deben almacenarse en tus **variables**.
* TypeScript espera que el tipo de dato de la variable coincida con el tipo del valor asignado inicialmente al momento de su declaración; esto se conoce como inferencia de tipos.
* La forma de un objeto describe, entre otras cosas, qué propiedades y métodos contiene o no contiene. TypeScript sabe no solo qué tipo tiene algo, sino también qué forma tiene ese tipo.
* Cuando no puede inferir un tipo, TypeScript considerará que una variable es de tipo `any`.
* Las anotaciones de tipo son pequeñas piezas de código que indican qué tipo debe tener una variable.

```ts
let youAreAwesome: boolean;
youAreAwesome = true;
```

---

