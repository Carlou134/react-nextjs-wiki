# Organización de Carpetas: Vertical Slice vs. Arquitectura Horizontal

## El problema que aparece cuando el proyecto crece

En un proyecto chico, casi cualquier forma de organizar las carpetas parece funcionar igual de bien. El problema no se nota en un proyecto de tres pantallas — se nota seis meses después, cuando el proyecto tiene treinta pantallas y modificar una sola funcionalidad te obliga a saltar entre diez carpetas distintas, cada una compartida con otras quince funcionalidades que no tienen nada que ver. En ese punto, la forma en la que organizaste el código deja de ser un detalle estético y se convierte en el factor que determina si el proyecto sigue siendo mantenible o no.

Hay dos enfoques principales para organizar el código fuente de una aplicación frontend: **horizontal** (por tipo técnico de archivo) y **vertical slice** (por capacidad de negocio). No son mutuamente excluyentes con todo lo demás que vimos en este wiki —los componentes, hooks y patrones siguen siendo los mismos—, pero determinan **dónde** vive cada pieza y qué tan fácil es encontrarla y modificarla.

-----

## Organización Horizontal

Es la forma más común de organizar un proyecto React por defecto, y probablemente la que ya usaste en varios de los ejemplos de este wiki: las carpetas de nivel superior representan un **tipo técnico** de archivo, sin importar a qué funcionalidad de negocio pertenece cada uno.

```
src/
├── components/
│   ├── UserCard.tsx
│   ├── ProductCard.tsx
│   ├── LoginForm.tsx
│   └── CheckoutForm.tsx
├── hooks/
│   ├── useAuth.ts
│   ├── useCart.ts
│   └── useProducts.ts
├── services/
│   ├── authService.ts
│   ├── cartService.ts
│   └── productService.ts
└── pages/
    ├── LoginPage.tsx
    ├── ProductsPage.tsx
    └── CheckoutPage.tsx
```

Esta estructura es sencilla de entender de entrada, porque cualquier desarrollador que conozca React sabe dónde buscar "un componente" o "un hook" sin necesidad de conocer el dominio de negocio del proyecto. El problema aparece cuando una funcionalidad específica —por ejemplo, todo lo relacionado con el carrito de compras— termina con piezas dispersas en `components/`, `hooks/`, `services/` y `pages/` **al mismo tiempo**, mezcladas ahí con piezas de otras funcionalidades completamente distintas (login, productos). Modificar o eliminar el carrito de compras exige entonces recorrer las cuatro carpetas, identificar cuáles archivos le pertenecen entre docenas de otros que no, y confiar en que no se te escapó ninguno.

-----

## Vertical Slice (organización por funcionalidad)

El enfoque de **Vertical Slice** invierte el criterio: en lugar de agrupar por tipo técnico, agrupa por **capacidad de negocio** (una *feature*). Cada carpeta de funcionalidad contiene todo lo que esa funcionalidad necesita —sus propios componentes, hooks, servicios y páginas— en un solo lugar:

```
src/
└── features/
    ├── auth/
    │   ├── components/
    │   │   └── LoginForm.tsx
    │   ├── hooks/
    │   │   └── useAuth.ts
    │   ├── services/
    │   │   └── authService.ts
    │   └── pages/
    │       └── LoginPage.tsx
    ├── cart/
    │   ├── components/
    │   │   └── CheckoutForm.tsx
    │   ├── hooks/
    │   │   └── useCart.ts
    │   ├── services/
    │   │   └── cartService.ts
    │   └── pages/
    │       └── CheckoutPage.tsx
    └── products/
        ├── components/
        │   └── ProductCard.tsx
        ├── hooks/
        │   └── useProducts.ts
        ├── services/
        │   └── productService.ts
        └── pages/
            └── ProductsPage.tsx
```

Dentro de cada módulo (`auth/`, `cart/`, `products/`) se repite la misma subestructura técnica (`components/`, `hooks/`, `services/`, `pages/`) que antes vivía a nivel de todo el proyecto — la diferencia es que ahora esa subestructura está **contenida dentro de una sola funcionalidad**, en lugar de mezclar ahí archivos de funcionalidades distintas. Modificar o eliminar el carrito de compras se reduce, en el caso ideal, a tocar únicamente lo que hay adentro de `features/cart/`.

Un heurístico simple para saber si la arquitectura de un proyecto está funcionando: **si para modificar una sola funcionalidad tenés que recorrer medio proyecto, la arquitectura está mal**. Vertical Slice apunta directamente a evitar ese síntoma.

-----

## `shared/` vs. `features/`

No todo el código pertenece a una única funcionalidad. Un botón genérico, un modal reutilizable, un hook como `useDebounce`, o un cliente HTTP configurado con los headers por defecto de la aplicación, no le pertenecen a `auth` ni a `cart` en particular — le pertenecen a **todo el proyecto**. Ese tipo de código va en una carpeta separada, generalmente llamada `shared/` (o `common/`):

```
src/
├── shared/
│   ├── components/
│   │   ├── Button.tsx
│   │   └── Modal.tsx
│   ├── hooks/
│   │   └── useDebounce.ts
│   └── lib/
│       └── httpClient.ts
└── features/
    ├── auth/
    ├── cart/
    └── products/
```

La pregunta que separa un archivo entre `shared/` y una `feature/` específica es: **¿este código tiene sentido fuera del contexto de esta funcionalidad?** Un `Button` genérico, sí — se puede usar en cualquier parte de la aplicación sin saber nada de autenticación ni de carritos. Un `LoginForm`, no — no tiene ningún sentido fuera del contexto de `auth`. Cuando la respuesta no es obvia, suele ser una señal de que ese código todavía está acoplado a una única funcionalidad y conviene dejarlo ahí hasta que un segundo caso de uso real demuestre que efectivamente es reutilizable — mover código a `shared/` de forma preventiva, "por si acaso se reutiliza algún día", suele generar abstracciones prematuras que nadie termina usando tal como se diseñaron.

-----

## El objetivo detrás de la organización

Más allá de la elección específica entre horizontal y vertical slice, el objetivo de fondo es siempre el mismo: **separar negocio, infraestructura y elementos reutilizables**, de forma que el proyecto sea entendible sin tener que sostener el mapa completo del código en la cabeza. Lógica de negocio (qué hace cada funcionalidad), infraestructura (cómo se conecta con el backend, cómo se configura el routing) y piezas reutilizables (componentes y hooks genéricos) son preocupaciones distintas, y mezclarlas en las mismas carpetas —sin ningún criterio de separación— es lo que hace que un proyecto se sienta cada vez más difícil de navegar a medida que crece, incluso cuando el código individual de cada archivo está bien escrito.

Vertical Slice no es la única forma válida de lograr esa separación, y para proyectos chicos la organización horizontal sigue siendo perfectamente razonable. Pero a medida que un proyecto crece en cantidad de funcionalidades independientes entre sí, agrupar por capacidad de negocio en lugar de por tipo técnico suele escalar mucho mejor, porque hace que el **límite de cada funcionalidad sea visible en la estructura de carpetas**, en lugar de vivir únicamente en la cabeza de quien escribió el código originalmente.
