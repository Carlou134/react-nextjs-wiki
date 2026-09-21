# Tipos de objeto avanzados: modelar, derivar y reutilizar

## En una frase

Los tipos de objeto describen la forma de los datos (`interface`, `type`, anidamiento, opcionales, firmas de índice), y los **utility types**, `keyof`, `typeof`, el acceso indexado, los genéricos y los *mapped types* permiten **derivar** tipos nuevos a partir de los existentes en lugar de reescribirlos.

-----

## Antes de empezar

Conviene que ya sepas:

* Cómo declarar tipos con `type`, uniones y literales: [Custom Types](05-Custom%20Types.md) y [Union Types](06-Union%20Types.md).
* Cómo reducir una unión con `typeof`, `in` o discriminantes: [Type Narrowing](07-Type%20Narrowing.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Miembro:** propiedad o método de un tipo de objeto.
* **Firma de índice (index signature):** declaración que describe un objeto cuyas claves no se conocen de antemano, pero sí el tipo de clave y de valor.
* **Utility type:** tipo genérico que ya viene en TypeScript y transforma otro tipo (`Partial`, `Pick`, `Omit`...).
* **Genérico:** tipo o función con un **parámetro de tipo** (`<T>`) que se fija al usarlo.
* **Mapped type:** tipo que recorre las claves de otro y construye uno nuevo a partir de ellas.

-----

## El problema

Un modelo de datos real crece y se repite:

```ts
interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

// Para el formulario de alta necesitas todo menos el id.
interface NewUser {
  name: string;
  email: string;
  role: 'admin' | 'user';
}

// Para el PATCH necesitas todo como opcional.
interface UserPatch {
  id?: number;
  name?: string;
  email?: string;
  role?: 'admin' | 'user';
}
```

Son tres declaraciones que describen lo mismo. Si añades un campo a `User`, hay que acordarse de actualizar las otras dos, y el compilador no avisa cuando se desincronizan. La solución es tener **una fuente de verdad** y derivar el resto.

-----

## Cómo funciona

### Modelar objetos: `interface` y `type`

Ambos describen la forma de un objeto:

```ts
type Mail = { postagePrice: number; address: string };

interface MailI {
  postagePrice: number;
  address: string;
}
```

Diferencias reales:

| | `interface` | `type` |
|---|---|---|
| Describe objetos | Sí | Sí |
| Describe uniones, primitivos, tuplas | No | Sí |
| Herencia | `extends` | intersección (`&`) |
| Declaraciones repetidas con el mismo nombre | Se **fusionan** (*declaration merging*) | Error |

Criterio práctico: `interface` para la forma de objetos que otros pueden extender (contratos, props de componentes); `type` para uniones, tuplas y tipos derivados. Lo importante es ser consistente en el proyecto.

### `implements`: una clase cumple un contrato

```ts
interface Robot {
  identify: (id: number) => void;
}

class OneSeries implements Robot {
  identify(id: number) {
    console.log(`beep, soy ${id.toFixed(2)}`);
  }

  answerQuestion() {
    console.log('42');
  }
}
```

`implements` solo **verifica** que la clase tenga los miembros del contrato. La clase puede tener más (`answerQuestion`). No copia tipos a la clase: los parámetros de `identify` se siguen tipando en la clase.

### Anidar y componer

Un tipo de objeto puede contener otros. Cuando el anidamiento se vuelve difícil de leer, o necesitas solo una parte, se nombran las piezas y se componen:

```ts
interface Version {
  versionNumber: number;
}

interface General {
  id: number;
  name: string;
  version: Version;
}

interface About {
  general: General;
}
```

Ahora `Version` se puede reutilizar sola.

### Extender: `extends` e intersección

```ts
interface Shape {
  color: string;
}

interface Square extends Shape {
  sideLength: number;
}

const mySquare: Square = { sideLength: 10, color: 'blue' };
```

Con `type` se logra lo mismo con `&`:

```ts
type Circle = Shape & { radius: number };
```

Diferencia: `extends` **falla al declarar** si un miembro es incompatible con el del padre. La intersección no falla: el miembro conflictivo se vuelve `never` y el error aparece después, al usarlo.

### Miembros opcionales y de solo lectura

```ts
interface Options {
  name: string;
  size?: string;          // opcional: string | undefined
  readonly id: number;    // no se puede reasignar
}

function listFile(options: Options) {
  return options.size ? `${options.name}: ${options.size}` : options.name;
}

listFile({ name: 'readme.txt', id: 1 });
```

`readonly` se comprueba solo en compilación; en ejecución la propiedad sigue siendo modificable.

### Firmas de índice

Sirven cuando no conoces los nombres de las claves, pero sí su tipo:

```ts
interface SolarEclipse {
  [latitude: string]: boolean;
}

const eclipse: SolarEclipse = {
  '40.712776': true,
  '40.417286': false,
};
```

`latitude` es solo un nombre descriptivo. Todos los demás miembros del tipo deben ser compatibles con el tipo del valor. `Record<string, boolean>` es la forma abreviada equivalente.

### `keyof` y `typeof`

* `keyof T` produce la **unión de las claves** de `T`.
* `typeof valor` (en posición de tipo) produce el **tipo de un valor** que ya existe.

```ts
type UserKey = keyof User;   // 'id' | 'name' | 'email' | 'role'

const defaultUser = { id: 0, name: 'Anónimo', role: 'user' as const };
type DefaultUser = typeof defaultUser;
// { id: number; name: string; role: 'user' }
```

Combinados sirven para derivar un tipo de una constante:

```ts
const ROLES = ['admin', 'user', 'guest'] as const;
type Role = (typeof ROLES)[number];   // 'admin' | 'user' | 'guest'
```

### Acceso indexado: `T[K]`

Extrae el tipo de una propiedad:

```ts
type UserName = User['name'];                 // string
type UserContact = User['name' | 'email'];    // string
type AnyUserValue = User[keyof User];         // string | number
```

Sobre arrays, `T[number]` es el tipo de sus elementos (ya se usó arriba con `ROLES`).

### Genéricos básicos

Un genérico parametriza un tipo o función. `K extends keyof T` restringe `K` a las claves reales de `T`:

```ts
interface ApiResponse<T> {
  data: T;
  error?: string;
}

const res: ApiResponse<User[]> = { data: [] };

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user: User = { id: 1, name: 'Ana', email: 'ana@mail.com', role: 'admin' };
const name = getProperty(user, 'name');   // string
// getProperty(user, 'phone');            // error: 'phone' no es clave de User
```

El tipo de retorno `T[K]` cambia según la clave: TypeScript sabe que `'name'` devuelve `string`.

### Utility types

Resuelven el problema del inicio. Todos parten de `User`:

```ts
type UserPatch    = Partial<User>;                 // todo opcional
type UserComplete = Required<UserPatch>;           // todo obligatorio
type UserPreview  = Pick<User, 'id' | 'name'>;     // solo esas claves
type NewUser      = Omit<User, 'id'>;              // todo menos esas claves
type RoleLabels   = Record<User['role'], string>;  // { admin: string; user: string }
type FrozenUser   = Readonly<User>;                // todo readonly
```

| Utility | Qué hace |
|---|---|
| `Partial<T>` | Vuelve opcionales todas las propiedades |
| `Required<T>` | Vuelve obligatorias todas las propiedades |
| `Pick<T, K>` | Se queda solo con las claves `K` |
| `Omit<T, K>` | Elimina las claves `K` |
| `Record<K, V>` | Objeto con claves `K` y valores `V` |
| `Readonly<T>` | Marca todas las propiedades como `readonly` |

`Record` con una unión de literales como clave **exige todas las claves**; con `string`, acepta cualquiera.

### Mapped types: cómo funcionan por dentro

Los utility types son mapped types. Recorren `keyof T` y construyen un objeto nuevo:

```ts
type MyPartial<T> = { [K in keyof T]?: T[K] };
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };
type MyRequired<T> = { [K in keyof T]-?: T[K] };   // -? quita la opcionalidad

type Flags<T> = { [K in keyof T]: boolean };
type UserFlags = Flags<User>;   // { id: boolean; name: boolean; ... }
```

Los modificadores `?` y `readonly` se pueden añadir (con `+` explícito o sin prefijo), y con `-` quitar. Casi nunca los escribirás a mano, pero conocerlos permite leer los tipos de librerías y crear los propios.

### `satisfies`: comprobar sin perder precisión

Desde TypeScript 4.9, `satisfies` verifica que un valor cumple un tipo **sin reemplazar** el tipo inferido del valor.

```ts
type Route = { path: string; title: string };

// Con anotación: se pierden las claves concretas
const a: Record<string, Route> = {
  home: { path: '/', title: 'Inicio' },
};
a.cualquierCosa;   // compila: para TypeScript, cualquier clave existe

// Con satisfies: se valida la forma y se conservan las claves
const routes = {
  home: { path: '/', title: 'Inicio' },
  about: { path: '/about', title: 'Acerca de' },
} satisfies Record<string, Route>;

routes.home.path;   // string
// routes.contact;  // error: la propiedad no existe
```

### Conexión con React

Estos tipos aparecen constantemente al tipar props y estado:

```tsx
import type { ComponentPropsWithoutRef } from 'react';

// Props de un botón propio: hereda todo lo de <button> y añade lo suyo
interface ButtonProps extends ComponentPropsWithoutRef<'button'> {
  variant: 'primary' | 'ghost';
}

function Button({ variant, children, ...rest }: ButtonProps) {
  return (
    <button className={variant} {...rest}>
      {children}
    </button>
  );
}
```

* `Partial<T>` para props opcionales o actualizaciones parciales de estado.
* `Omit` / `Pick` para que un componente reciba solo una parte de un modelo.
* `Record` para diccionarios en estado (`Record<number, User>`).
* Genéricos para componentes y Hooks reutilizables (`useState<T>`, `useFetch<T>`).

-----

## Ejemplo completo

Un formulario de alta de usuario que deriva todos sus tipos de `User`:

```tsx
import { useState, type FormEvent } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

type UserDraft = Omit<User, 'id'>;
type UserErrors = Partial<Record<keyof UserDraft, string>>;

const emptyDraft: UserDraft = { name: '', email: '', role: 'user' };

interface UserFormProps {
  initial?: Partial<UserDraft>;
  onSave: (user: UserDraft) => void;
}

export function UserForm({ initial, onSave }: UserFormProps) {
  const [draft, setDraft] = useState<UserDraft>({ ...emptyDraft, ...initial });
  const [errors, setErrors] = useState<UserErrors>({});

  // K se infiere de la clave; el valor debe coincidir con el tipo de ESA clave
  function update<K extends keyof UserDraft>(key: K, value: UserDraft[K]) {
    setDraft((prev) => ({ ...prev, [key]: value }));
  }

  function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault();

    const next: UserErrors = {};
    if (draft.name.trim() === '') next.name = 'El nombre es obligatorio';
    if (!draft.email.includes('@')) next.email = 'El email no es válido';

    setErrors(next);
    if (Object.keys(next).length === 0) onSave(draft);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={draft.name}
        onChange={(e) => update('name', e.target.value)}
        placeholder="Nombre"
      />
      {errors.name && <p>{errors.name}</p>}

      <input
        value={draft.email}
        onChange={(e) => update('email', e.target.value)}
        placeholder="Email"
      />
      {errors.email && <p>{errors.email}</p>}

      <button type="submit">Guardar</button>
    </form>
  );
}
```

Puntos clave:

1. `User` es la única fuente de verdad. `UserDraft` y `UserErrors` se derivan y se mantienen sincronizados solos.
2. `UserErrors` tiene las mismas claves que el formulario, todas opcionales. Si agregas un campo a `User`, aparece también en los errores.
3. `update('name', 5)` no compila: `UserDraft['name']` es `string`. Los genéricos con `keyof` evitan escribir un handler por campo sin perder seguridad.
4. `initial` es `Partial<UserDraft>`: quien use el componente puede precargar solo algunos campos.

-----

## Errores comunes

### 1. Esperar que `Omit` avise de una clave inexistente

```ts
type Wrong = Omit<User, 'idd'>;   // compila, y no elimina nada
```

**Por qué pasa:** `Omit<T, K>` declara `K extends keyof any`, así que acepta cualquier clave.
**Solución:** si necesitas que valide, define tu propio `type StrictOmit<T, K extends keyof T> = Omit<T, K>`. (`Pick<T, Exclude<keyof T, K>>` tampoco valida, porque `Exclude` no comprueba que `K` exista.)

### 2. Creer que `Readonly` y `Partial` son profundos

```ts
const u: Readonly<{ tags: string[] }> = { tags: [] };
u.tags.push('x');   // compila
```

**Por qué pasa:** los utility types actúan solo sobre el **primer nivel**. `tags` no se puede reasignar, pero el array sigue siendo mutable.
**Solución:** para arrays, `readonly string[]` o `ReadonlyArray<string>`; para anidados, un tipo recursivo propio.

### 3. Firma de índice incompatible con otros miembros

```ts
interface Config {
  [key: string]: string;
  retries: number;   // error
}
```

**Por qué pasa:** la firma de índice afirma que **todas** las propiedades son `string`, y `retries` es `number`.
**Solución:** amplía el valor de la firma (`string | number`) o separa los datos fijos de los dinámicos en dos propiedades.

### 4. Confiar en que una firma de índice garantiza que la clave existe

```ts
const eclipse: Record<string, boolean> = {};
const visible = eclipse['40.7'];   // tipo boolean, valor undefined
```

**Por qué pasa:** el tipo dice que toda clave devuelve `boolean`, aunque no exista.
**Solución:** activa `noUncheckedIndexedAccess` en `tsconfig.json` (el acceso pasa a `boolean | undefined`) o usa `Partial<Record<string, boolean>>`.

### 5. Asumir que `Object.keys` devuelve `(keyof T)[]`

```ts
Object.keys(user).forEach((k) => user[k]);   // error: k es string
```

**Por qué pasa:** un objeto puede tener más claves en ejecución que las declaradas en su tipo, así que `Object.keys` devuelve `string[]`.
**Solución:** si sabes que el objeto no tiene claves extra, haz un cast acotado (`as (keyof User)[]`) o recorre una lista de claves definida con `as const`.

-----

## Cuándo sí y cuándo no

**Usa tipos derivados cuando:**

* Varias formas de datos comparten campos con un modelo base (alta, edición, vista previa).
* Quieres que un cambio en el modelo se propague sin editar cada copia.
* Necesitas claves o valores de una constante como tipo (`typeof` + `as const`).

**No los uses cuando:**

* Derivar hace el tipo ilegible: una forma pequeña y estable se lee mejor escrita explícitamente.
* Encadenas muchos utility types (`Partial<Omit<Pick<...>>>`). Pon nombre a cada paso intermedio.
* Una firma de índice esconde un dominio que sí es cerrado. Si las claves se conocen, usa un `Record` con unión de literales o una interface.

-----

## Resumen en 5 líneas

1. `interface` y `type` describen objetos; `interface` se extiende con `extends`, `type` con `&` y además admite uniones y tuplas.
2. Los opcionales (`?`), `readonly` y las firmas de índice (`[k: string]: V`) ajustan la forma de un objeto.
3. `keyof T` da las claves, `typeof valor` el tipo de un valor y `T[K]` el tipo de una propiedad.
4. `Partial`, `Required`, `Pick`, `Omit`, `Record` y `Readonly` derivan tipos desde un modelo único; por dentro son mapped types.
5. `satisfies` valida un valor sin perder su tipo inferido, y los genéricos con `K extends keyof T` dan handlers seguros en React.

-----

## Para profundizar

<details>
<summary>Declaration merging: por qué existe en interface y no en type</summary>

Dos `interface` con el mismo nombre en el mismo ámbito se combinan en una sola:

```ts
interface Settings { theme: string }
interface Settings { fontSize: number }

const s: Settings = { theme: 'dark', fontSize: 14 };
```

Es la base para ampliar tipos de librerías (por ejemplo, añadir propiedades a un tipo global). Un `type` con nombre repetido produce el error "Duplicate identifier". Es una razón legítima para elegir `interface` en tipos públicos que otros deban poder ampliar.

</details>

<details>
<summary>Remapeo de claves en mapped types</summary>

Con `as` dentro de un mapped type se pueden **renombrar** o **filtrar** claves:

```ts
type Getters<T> = {
  [K in keyof T & string as `get${Capitalize<K>}`]: () => T[K];
};

type UserGetters = Getters<{ name: string; age: number }>;
// { getName: () => string; getAge: () => number }
```

`Capitalize` es otro utility type, este actúa sobre cadenas de texto. Para filtrar, se remapea la clave a `never`.

</details>

<details>
<summary>Otros utility types útiles</summary>

* `ReturnType<typeof fn>`: tipo que devuelve una función.
* `Parameters<typeof fn>`: tupla con los tipos de sus parámetros.
* `Awaited<Promise<T>>`: `T`, con la promesa "desenvuelta".
* `Exclude<U, X>` y `Extract<U, X>`: quitan o dejan miembros de una **unión**.
* `NonNullable<T>`: quita `null` y `undefined` de un tipo.
* `ComponentProps<typeof Componente>`: props de un componente de React ya existente.

</details>

-----

## En entrevista

### Respuesta corta (junior)

`interface` y `type` sirven para describir la forma de un objeto. Los utility types, como `Partial`, `Pick` y `Omit`, permiten crear tipos nuevos a partir de uno existente sin repetirlo. Por ejemplo, `Omit<User, 'id'>` describe un usuario sin `id`, útil para un formulario de alta.

### Respuesta ampliada (semi-senior)

* **`interface` vs `type`:** ambos describen objetos. `interface` admite `extends` y *declaration merging*; `type` admite uniones, tuplas, primitivos y mapped types. `extends` detecta conflictos al declarar; `&` los convierte en `never`.
* **Una fuente de verdad:** los tipos derivados (`Partial`, `Pick`, `Omit`, `Record`) evitan duplicar modelos y hacen que un cambio se propague.
* **`keyof`, `typeof`, `T[K]`:** permiten obtener claves, tipos de valores y tipos de propiedades. Con `as const` derivan uniones de literales desde datos reales.
* **Genéricos con restricciones:** `K extends keyof T` liga la clave con el tipo del valor (`T[K]`), lo que da APIs seguras como el `update(key, value)` de un formulario.
* **Mapped types:** los utility types son `{ [K in keyof T]: ... }` con modificadores (`?`, `readonly`, `-?`).
* **`satisfies`:** valida sin ensanchar el tipo inferido, a diferencia de una anotación.
* **Límites:** los utility types son superficiales; `Omit` no valida claves; una firma de índice miente sobre claves ausentes salvo con `noUncheckedIndexedAccess`; los tipos desaparecen en ejecución.

### Preguntas frecuentes de seguimiento

**1. ¿Cuándo usas `interface` y cuándo `type`?**
`interface` para contratos de objetos que se extienden o amplían (props, modelos, clases); `type` para uniones, tuplas y tipos derivados. Para objetos simples la diferencia práctica es pequeña, por lo que importa más la consistencia.

**2. ¿Qué diferencia hay entre `Pick` y `Omit`?**
`Pick<T, K>` conserva solo las claves indicadas; `Omit<T, K>` conserva todas menos esas. Se usa el que exprese la lista más corta. `Pick` valida que las claves existan; `Omit` no.

**3. ¿Qué hace `keyof typeof obj`?**
`typeof obj` obtiene el tipo del objeto y `keyof` la unión de sus claves. Sirve para tipar un parámetro que solo acepta las claves de una constante, sin repetirlas a mano.

**4. ¿Para qué sirve `satisfies` frente a una anotación de tipo?**
La anotación fija el tipo de la variable y descarta el inferido (se pierden claves concretas, por ejemplo). `satisfies` solo comprueba que el valor cumple el tipo y conserva el inferido.

**5. ¿`Readonly<T>` hace el objeto inmutable?**
No en ejecución: solo impide reasignar propiedades en compilación y solo en el primer nivel. Los objetos o arrays anidados siguen siendo mutables.

**6. ¿Cómo tipas un componente que envuelve un `<button>`?**
Extiendes las props nativas con `ComponentPropsWithoutRef<'button'>` y añades las propias, o usas `Omit` para retirar las que quieras controlar tú. Así el componente acepta `onClick`, `disabled`, etc. sin declararlas.

-----

## Siguiente lección

Con los fundamentos de TypeScript cubiertos, es momento de aplicarlos en React: [Intro to JSX](../01-fundamentos-jsx/01-Intro%20to%20JSX.md).
