# Tipado de Estados y Funciones

## Why Type State and Functions?

En la lección anterior aprendimos a tipar las props que recibe un componente. Pero las props no son el único lugar donde manejamos datos: cada vez que usamos `useState()` estamos creando una nueva pieza de estado interno, y ese estado también necesita reglas claras sobre qué tipo de dato puede contener. Lo mismo aplica a las funciones que escribimos dentro y fuera de nuestros componentes: cada una debería declarar explícitamente qué tipo de datos recibe y qué tipo de dato devuelve.

Tipar estado y funciones sigue exactamente el mismo espíritu que tipar props: hacer explícitas las reglas de nuestros datos para que TypeScript pueda detectar errores antes de que el código se ejecute.

-----

## Typing useState

Cuando llamamos a `useState()` con un valor inicial, TypeScript generalmente puede **inferir** el tipo del estado a partir de ese valor inicial. Por ejemplo, si escribimos `useState(0)`, TypeScript entiende automáticamente que el estado es de tipo `number`, sin que tengamos que decírselo explícitamente.

Sin embargo, hay situaciones en las que conviene ser explícitos, usando la sintaxis de **tipos genéricos** de `useState<T>()`:

```tsx
const [count, setCount] = useState<number>(0);
```

Aquí, `<number>` le indica a TypeScript, sin ambigüedad, que `count` va a ser siempre un número a lo largo de toda la vida del componente. A partir de esta declaración, cualquier intento de actualizar el estado con un valor que no sea numérico va a ser marcado como un error:

```tsx
setCount(count + "hola"); // Error: el argumento no es de tipo 'number'
```

TypeScript rechaza esta línea porque sumar un `number` con un `string` produce un resultado (`"5hola"`, un `string`) que no es compatible con lo que `setCount` espera recibir según cómo tipamos el estado.

Ser explícito con el genérico es especialmente útil cuando el valor inicial no alcanza para que TypeScript infiera correctamente el tipo que vas a necesitar más adelante. Un caso típico es un estado que empieza en `null` pero que eventualmente va a contener un objeto:

```tsx
type User = {
  id: number;
  name: string;
};

const [user, setUser] = useState<User | null>(null);
```

Si escribiéramos simplemente `useState(null)`, TypeScript infiere que el tipo del estado es `null` para siempre, y no nos dejaría asignarle nunca un objeto `User`. Al declarar explícitamente `useState<User | null>(null)`, le decimos a TypeScript que este estado puede ser, en distintos momentos de la vida del componente, o bien `null` (mientras el usuario todavía no cargó) o bien un objeto con la forma `User`.

-----

## Typing Function Parameters and Return Values

De la misma manera que tipamos el estado, deberíamos tipar explícitamente los parámetros y el valor de retorno de nuestras funciones:

```ts
const greet = (name: string): string => {
  return `Hola ${name}`;
};
```

Esta firma establece dos reglas: `greet` solo acepta un argumento de tipo `string`, y siempre devuelve un `string`. Si en algún lugar de la aplicación intentáramos llamar `greet(42)`, TypeScript marcaría el error inmediatamente, porque `42` es un `number`, no un `string`.

Tipar el valor de retorno no es solo una formalidad: actúa como una **verificación adicional** sobre la implementación de la función. Si por error escribiéramos una línea que devuelve un número en lugar de un texto, TypeScript señalaría el problema dentro de la propia función, sin necesidad de esperar a que algún otro lugar del código intente usar mal el resultado.

-----

## Why Avoid `any`

TypeScript incluye un tipo especial llamado `any`, que básicamente desactiva la verificación de tipos para ese valor en particular:

```ts
const greet = (name: any): any => {
  // ...
};
```

Usar `any` es, en la práctica, decirle a TypeScript "no verifiques nada acá, confío en que yo sé lo que estoy haciendo". El problema es que, al hacer esto, perdés exactamente los tres beneficios que motivan usar TypeScript en primer lugar:

* **Validación**: TypeScript deja de poder avisarte si le pasás un dato con la forma incorrecta.
* **Autocompletado**: el editor ya no puede sugerirte las propiedades o métodos disponibles, porque no tiene ninguna información sobre la forma real del dato.
* **Seguridad**: cualquier error de tipos que hubiera existido en ese código queda oculto hasta que el programa se ejecuta, que es exactamente el escenario que TypeScript existe para evitar.

En términos prácticos, cada `any` que aparece en un proyecto TypeScript es un punto ciego: una zona del código donde, silenciosamente, perdiste todas las garantías que el resto del sistema de tipos te está dando. Cuando no sabés (o todavía no sabés) qué forma exacta tiene un dato, existen alternativas más seguras que `any`, como `unknown`, que obliga a verificar el tipo del valor antes de operar con él; pero para la enorme mayoría de los casos, lo correcto es simplemente tomarse el tiempo de definir el tipo real que corresponde.

-----

## Putting It Together

Tipar estado y funciones consistentemente convierte a TypeScript en una capa de verificación activa a lo largo de toda tu aplicación: el estado declara qué forma de dato puede contener en cada momento, y las funciones declaran qué reciben y qué devuelven. Cuando estas dos piezas están bien tipadas, TypeScript puede rastrear el flujo completo de un dato (desde que nace en un `useState`, pasa por una función, y termina siendo usado en el JSX) y avisarte apenas algo deja de encajar, mucho antes de que ese error llegue a un usuario real.
