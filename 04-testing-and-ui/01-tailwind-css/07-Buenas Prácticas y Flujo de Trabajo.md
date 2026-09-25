# Buenas prácticas con Tailwind: componentes, variantes y flujo de trabajo

## En una frase

Tailwind no elimina la repetición por sí mismo: la elimina el **componente**. Las buenas prácticas consisten en que cada componente sea el único lugar donde viven sus clases, en combinar variantes sin conflictos (`cn`), en mantener las cadenas legibles y en revisar de forma sistemática todo código que no escribiste tú (plantillas copiadas o generadas por IA).

-----

## Antes de empezar

Conviene que ya conozcas:

* Cómo funciona el enfoque utility-first y las clases básicas: [Fundamentos de Tailwind](01-Fundamentos%20de%20Tailwind.md).
* Por qué las clases deben escribirse completas en el código: [Cómo Tailwind detecta las clases](03-C%C3%B3mo%20Tailwind%20Detecta%20las%20Clases.md).
* Cómo se definen tokens de diseño en el tema: [Tema, modo oscuro y plugins](06-Tema%2C%20Modo%20Oscuro%20y%20Plugins.md).

Palabras usadas en esta nota (también están en el [Glosario](Glosario.md)):

* **Variante (de componente):** versión de un componente que cambia su aspecto según una prop, por ejemplo un botón `primary` o `secondary`. No confundir con las *variantes de Tailwind* (`hover:`, `md:`), que aplican una clase solo bajo cierta condición.
* **Design token:** valor de diseño con nombre (un color, un espaciado, una tipografía) definido en un solo lugar. En Tailwind v4 se declaran como variables de tema con `@theme`.
* **Conflicto de clases:** dos clases que definen la misma propiedad CSS sobre el mismo elemento, como `p-2` y `p-4`.
* **`@apply`:** directiva de Tailwind que copia las declaraciones de varias utilidades dentro de una regla CSS propia.

-----

## El problema

El costo principal de Tailwind es que el marcado acumula cadenas largas de clases:

```tsx
<button className="inline-flex items-center justify-center rounded-md bg-blue-600 px-4 py-2 font-semibold text-white transition-colors hover:bg-blue-700 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600 disabled:pointer-events-none disabled:opacity-50">
  Guardar
</button>
```

Si esa cadena se copia en veinte lugares aparecen tres problemas:

1. **Repetición:** cambiar el azul de los botones obliga a editar veinte archivos.
2. **Deriva:** las copias se editan por separado y, con el tiempo, dejan de ser iguales.
3. **Ilegibilidad:** nadie distingue en una revisión de código qué clase es intencional y cuál es un resto.

Tailwind no resuelve esto con una regla propia. Su documentación lo trata como un problema de abstracción y recomienda el mecanismo que ya tiene tu framework: un **componente** (o un *template partial* si usas un lenguaje de plantillas).

-----

## Cómo funciona

### Pensar en componentes, no en pantallas

La documentación oficial ("Reusing styles") distingue tres casos según dónde ocurre la repetición:

* **Elementos en un bucle:** la lista de clases se escribe una sola vez dentro del `map`. No hay duplicación real que resolver.
* **Duplicación en un solo archivo:** edítala con cursores múltiples del editor. Si puedes cambiar todas las copias a la vez, una abstracción adicional no aporta.
* **Duplicación entre archivos:** crea un **componente**. Es la estrategia recomendada en React, Vue o Svelte.

Regla práctica: **el componente es el único lugar donde viven las clases** de esa pieza de interfaz. Se reutiliza importándolo, no copiando su cadena.

```tsx
function Card({ children }: { children: React.ReactNode }) {
  return (
    <div className="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
      {children}
    </div>
  );
}
```

Si mañana cambia el radio de las tarjetas, se modifica en un único archivo. El mecanismo de reutilización no es una clase CSS, es un componente de React, igual que en los patrones de composición.

### `@apply` con moderación

`@apply` inserta las declaraciones de varias utilidades dentro de una regla CSS propia:

```css
.select2-dropdown {
  @apply rounded-b-lg shadow-md;
}
```

La documentación oficial lo presenta para un caso concreto: escribir CSS propio, por ejemplo para **sobrescribir los estilos de una librería de terceros**, sin abandonar tus design tokens ni la sintaxis de las utilidades.

Como sustituto de un componente tiene costos:

* Reintroduce nombres de clase inventados (`.btn`) y archivos CSS que mantener, justo lo que utility-first quería evitar.
* Reintroduce el problema de nombrar y de la cascada: el orden entre tu clase y otras utilidades vuelve a importar.
* En archivos CSS Modules, en bloques `<style>` de Vue o Svelte, o en cualquier CSS que no se procese junto con tu hoja principal, hay que importar el tema con `@reference` antes de usar `@apply`. Sin `@reference`, esos archivos no conocen tus utilidades ni tu tema.

En un proyecto con componentes casi siempre es mejor extraer un componente. Reserva `@apply` para HTML plano, contenido que no controlas (por ejemplo, HTML que viene de un CMS) o estilos de librerías externas.

### Combinar clases condicionales

Un componente con variantes necesita elegir clases según props o estado. La forma más simple respeta lo que vimos sobre detección: un **mapa de clases completas**.

```tsx
const variants = {
  primary: "bg-blue-600 text-white hover:bg-blue-700",
  secondary: "bg-gray-200 text-gray-900 hover:bg-gray-300",
} as const;
```

La documentación oficial muestra este mismo patrón: mapear props a nombres de clase completos y estáticos. Nunca construyas la clase con una plantilla de texto (`` `bg-${color}-600` ``): Tailwind lee tu código como texto plano y no ejecuta ni interpola el JavaScript, así que esa clase no se genera.

Cuando la lógica crece intervienen dos utilidades pequeñas:

* **`clsx`** arma un string de clases a partir de valores condicionales (strings, objetos, arrays). Su README lo describe como una utilidad diminuta para construir strings de `className` de forma condicional.

  ```ts
  clsx("base", isActive && "font-bold", { "opacity-50": disabled });
  ```

* **`tailwind-merge`** (función `twMerge`) resuelve **conflictos** entre clases de Tailwind: cuando dos clases afectan la misma propiedad, conserva la última.

  ```ts
  twMerge("px-2 py-1 bg-red-500", "p-3 bg-blue-500");
  // "p-3 bg-blue-500"
  ```

`twMerge` hace falta porque el orden dentro de `className` no decide quién gana. En CSS, entre dos reglas de igual especificidad gana la que aparece **después en la hoja de estilos**, no la que aparece después en el atributo. La documentación de Tailwind lo resume así: en general, nunca pongas dos clases en conflicto sobre el mismo elemento. Un componente con `className` como prop puede recibir una clase que choque con las suyas; `twMerge` elimina la clase perdedora antes de renderizar.

Ambas se combinan en una función auxiliar, habitualmente llamada `cn`. Es el patrón que popularizó shadcn/ui:

```ts
// lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

`clsx` aplana y filtra los valores condicionales; `twMerge` resuelve los conflictos del resultado. Nota: la versión 3.x de `tailwind-merge` es para Tailwind v4; para proyectos en Tailwind v3 se usa `tailwind-merge` 2.6.0.

### Legibilidad de las cadenas largas

Las cadenas largas son el costo de Tailwind. Tres hábitos las mantienen manejables:

* **Orden consistente y automático.** El plugin oficial `prettier-plugin-tailwindcss` ordena las clases según el orden recomendado por Tailwind al formatear, así que todo el equipo las lee igual. En Tailwind v4 debe indicarse la hoja de estilos de entrada con la opción `tailwindStylesheet`. Con la opción `tailwindFunctions` también ordena las clases dentro de llamadas como `cn(...)` o `clsx(...)`.
* **Agrupar por propósito** cuando la cadena es muy larga: usa una cadena por grupo (layout, tipografía, estados) dentro de `cn(...)`, en lugar de una sola línea de 200 caracteres.
* **Extraer el componente** cuando la misma cadena se repite. Si copiaste veinte clases dos veces, ya toca un componente.

### Accesibilidad

Tailwind facilita la accesibilidad, pero no la garantiza. Cuatro hábitos básicos:

* **HTML semántico primero.** Usa `<button>`, `<a>`, `<label>`. Un `<div onClick>` con clases de botón no recibe foco ni responde al teclado.
* **Contraste.** Elige combinaciones legibles. WCAG AA pide, como referencia, una relación de contraste de 4.5:1 para texto normal. Una clase no asegura eso; verifícalo con una herramienta de contraste.
* **Foco visible.** Quien navega con teclado necesita ver qué elemento está activo. La variante `focus-visible:` aplica estilos solo cuando el foco viene del teclado (por ejemplo, `focus-visible:outline-2`). Quitar el contorno sin ofrecer un reemplazo deja la interfaz inutilizable con teclado.
* **Texto solo para lectores de pantalla.** La clase `sr-only` oculta un elemento visualmente pero lo deja disponible para tecnologías de asistencia, por ejemplo para nombrar un botón que solo tiene un ícono. `not-sr-only` deshace ese efecto.

```tsx
<button type="button">
  <TrashIcon aria-hidden="true" className="size-4" />
  <span className="sr-only">Eliminar tarea</span>
</button>
```

Tailwind también ofrece variantes ligadas a preferencias y atributos ARIA (`motion-reduce:`, `aria-disabled:`), útiles para respetar la preferencia del usuario de reducir animaciones.

### Reutilizar diseños ya hechos

Es legítimo partir de una colección de componentes ya armados con Tailwind. Un ejemplo es HyperUI, un conjunto de componentes gratuitos (licencia MIT) que se copian y pegan en el proyecto.

Lo importante es lo que haces después de pegar:

1. **Adaptar los valores a tu tema.** Sustituye colores y espaciados arbitrarios por tus tokens (`bg-brand` si definiste `--color-brand` en `@theme`).
2. **Extraerlo a un componente** si se va a repetir.
3. **Revisar la versión de Tailwind** que asume el snippet (ver la siguiente sección).
4. **Revisar accesibilidad**: contraste, foco, etiquetas.

Un componente copiado y no adaptado es el origen más común de una interfaz inconsistente.

### Trabajar con asistentes de IA

Los asistentes generan código de Tailwind rápido, pero su salida es un **borrador**. Revisa:

* **La versión de Tailwind.** Es habitual que el código use sintaxis de v3 aunque tu proyecto esté en v4. Diferencias frecuentes, según la guía oficial de actualización:
  * `@tailwind base; @tailwind components; @tailwind utilities;` pasó a `@import "tailwindcss";`.
  * `tailwind.config.js` ya no se detecta automáticamente; se carga con `@config` si aún se necesita. La configuración nueva va en CSS con `@theme`.
  * Escalas renombradas: `shadow-sm` pasó a `shadow-xs` y `shadow` a `shadow-sm`; `rounded-sm` a `rounded-xs` y `rounded` a `rounded-sm`; lo mismo con `blur`.
  * `outline-none` pasó a `outline-hidden`.
  * `ring` ahora vale 1px (antes 3px); `ring-3` da el ancho anterior.
  * `bg-gradient-to-r` pasó a `bg-linear-to-r`.
* **Clases dinámicas.** Plantillas de texto como `` `bg-${color}-500` `` no se generan.
* **Consistencia con tu proyecto.** Que use tus tokens y tus componentes existentes en lugar de estilos sueltos.
* **Accesibilidad.** Contraste, foco visible, etiquetas.

Regla general: si no entiendes qué hace una clase del código recibido, búscala en la documentación antes de aceptarla. Tailwind bien usado sigue dependiendo de entender CSS.

-----

## Ejemplo completo

Un `Button` con variantes y tamaños, con foco visible, estado deshabilitado y posibilidad de extenderlo con `className`. Usa el helper `cn` definido arriba.

```tsx
// components/Button.tsx
import { cn } from "@/lib/utils";

const variants = {
  primary: "bg-blue-600 text-white hover:bg-blue-700",
  secondary: "bg-gray-200 text-gray-900 hover:bg-gray-300",
  danger: "bg-red-600 text-white hover:bg-red-700",
} as const;

const sizes = {
  sm: "px-3 py-1.5 text-sm",
  md: "px-4 py-2 text-base",
} as const;

type ButtonProps = React.ComponentProps<"button"> & {
  variant?: keyof typeof variants;
  size?: keyof typeof sizes;
};

export function Button({
  variant = "primary",
  size = "md",
  type = "button",
  className,
  ...props
}: ButtonProps) {
  return (
    <button
      type={type}
      className={cn(
        "inline-flex items-center justify-center rounded-md font-semibold transition-colors",
        "focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600",
        "disabled:pointer-events-none disabled:opacity-50",
        variants[variant],
        sizes[size],
        className,
      )}
      {...props}
    />
  );
}
```

```tsx
<Button>Guardar</Button>
<Button variant="secondary" size="sm">Cancelar</Button>
<Button variant="danger" className="px-8">Eliminar</Button>
```

Puntos clave:

1. Cada variante y tamaño es un **mapa de clases completas**, detectables por Tailwind.
2. `cn` une la base, la variante, el tamaño y el `className` externo. En el último botón, `px-8` reemplaza a `px-4` gracias a `twMerge`; sin él, ambos quedarían en el DOM y decidiría el orden de la hoja de estilos.
3. `React.ComponentProps<"button">` hereda todas las props nativas (`onClick`, `disabled`, `aria-*`), así que no hay que declararlas una por una.
4. `type = "button"` evita que el botón envíe un formulario por accidente, porque el valor por defecto de `<button>` dentro de un `<form>` es `submit`.
5. La accesibilidad está en el componente: foco visible y estado deshabilitado se aplican a todas las instancias.

-----

## En React

* **El componente es la unidad de reutilización.** Extrae cuando una cadena se repite entre archivos, no antes: dos copias dentro de un mismo archivo no justifican una abstracción.
* **Expón `className` como prop** en los componentes de presentación y combínala con `cn`, para que quien use el componente pueda ajustar márgenes o anchos sin editar su código.
* **Variantes como props tipadas.** Una unión de literales (`"primary" | "secondary"`) hace que TypeScript rechace variantes inexistentes. `keyof typeof variants` mantiene tipo y mapa sincronizados.
* **Server Components.** Los componentes que solo devuelven marcado con clases funcionan sin `'use client'`, porque Tailwind produce CSS estático en build y no necesita JavaScript en el navegador. `cn` es una función pura y también funciona en el servidor.
* **Componentes copiados a tu repositorio.** shadcn/ui usa este mismo enfoque: los componentes viven en tu proyecto, están construidos con Tailwind y combinan clases con `cn`. Es la lección siguiente.

-----

## Errores comunes

### 1. Copiar cadenas de clases entre archivos

**Qué pasa:** el mismo botón o tarjeta aparece en veinte lugares con pequeñas diferencias.
**Por qué:** copiar es más rápido que extraer, y nada en Tailwind te obliga a abstraer.
**Arreglo:** extrae un componente cuando la duplicación cruce archivos.

### 2. Construir clases dinámicamente

```tsx
<div className={`bg-${color}-500`} />   // no se genera
```

**Qué pasa:** el estilo no aparece en el build, aunque a veces funcione en desarrollo.
**Por qué:** Tailwind lee el código como texto plano; `bg-red-500` nunca aparece completo en el archivo.
**Arreglo:** un mapa con clases completas (`{ red: "bg-red-500", blue: "bg-blue-500" }`).

### 3. Contar con el orden de las clases en `className`

```tsx
<div className="p-2 p-4" />   // no garantiza que gane p-4
```

**Qué pasa:** gana una clase distinta de la que esperabas.
**Por qué:** el orden de la hoja de estilos decide, no el del atributo.
**Arreglo:** no escribas dos clases en conflicto; para componentes con `className` externo, usa `cn` (`twMerge`).

### 4. Abusar de `@apply`

**Qué pasa:** aparecen clases `.btn`, `.card`, `.input` en archivos CSS paralelos a los componentes.
**Por qué:** se reproduce el CSS tradicional con otra sintaxis.
**Arreglo:** extrae componentes; deja `@apply` para contenido que no controlas o CSS de terceros.

### 5. Quitar el foco sin reemplazo

```tsx
<button className="outline-none">...</button>   // sin foco visible
```

**Qué pasa:** el usuario de teclado no ve dónde está.
**Por qué:** se eliminó el indicador del navegador sin poner otro.
**Arreglo:** añade `focus-visible:outline-2 focus-visible:outline-blue-600` (o `focus-visible:ring-2`).

### 6. Aceptar código de IA o de plantillas sin revisarlo

**Qué pasa:** aparece sintaxis de v3 en un proyecto v4, o colores que no son de tu tema.
**Por qué:** el código es un borrador que asume otro contexto.
**Arreglo:** revisa la lista de la sección "Trabajar con asistentes de IA" antes de integrarlo.

### 7. "No se ven los estilos"

Revisa en orden, de lo más común a lo menos común:

1. **¿El CSS se genera y se carga?** Debe importarse el CSS de entrada con `@import "tailwindcss";` y estar activo el plugin (`@tailwindcss/vite` o `@tailwindcss/postcss`) o el CLI con `--watch`. Comprueba que el `import` o `<link>` apunte al archivo correcto.
2. **¿La clase está bien escrita?** Un error de tipeo no da error: simplemente no se aplica.
3. **¿La clase se construye dinámicamente?** Debe estar completa en el código.
4. **¿Mezclas versiones?** Una directiva o clase de v3 en un proyecto v4, o al revés.
5. **¿El archivo está fuera de la detección?** Tailwind no escanea lo ignorado por `.gitignore` ni `node_modules`. Añádelo con `@source`.
6. **¿Usas `@apply` en un CSS Module o bloque `<style>` sin `@reference`?** Falla al no conocer tus utilidades y tu tema.
7. **¿Otra regla pisa la clase?** Inspecciona el elemento en DevTools y mira qué declaración gana y por qué.
8. **¿Reiniciaste el servidor?** Sobre todo tras cambiar plugins o la configuración.

-----

## Cuándo sí y cuándo no

**Sí:**

* Extrae un componente cuando la misma interfaz se repite entre archivos.
* Usa `cn` (`clsx` + `twMerge`) en componentes que reciben `className` o tienen variantes con posibles conflictos.
* Usa un mapa de clases completas para variantes simples.
* Usa `@apply` para CSS de terceros o contenido que no controlas.
* Usa plantillas y colecciones como punto de partida, y adáptalas.

**No:**

* No extraigas por adelantado: un bucle o dos copias en un archivo no lo requieren.
* No metas `twMerge` por costumbre si tus componentes no reciben `className` externo ni generan conflictos: es una dependencia y una llamada de JavaScript en runtime que no aportan nada.
* No uses `@apply` como sustituto sistemático de los componentes.
* No confíes en el orden del `className` para resolver conflictos.
* No aceptes código generado sin comprobar versión, tokens y accesibilidad.

-----

## Resumen en 5 líneas

1. La reutilización en Tailwind ocurre en el **componente**: cada componente es el único lugar donde viven sus clases.
2. `@apply` sirve para CSS de terceros o contenido que no controlas; en un proyecto con componentes, casi siempre conviene extraer un componente.
3. Para variantes usa un **mapa de clases completas** y, si hay conflictos o `className` externo, `cn` (`clsx` + `twMerge`).
4. Mantén el orden con `prettier-plugin-tailwindcss` y cuida la accesibilidad: HTML semántico, contraste, `focus-visible` y `sr-only`.
5. Plantillas y código de IA son borradores: revisa versión de Tailwind, clases dinámicas, tokens del tema y accesibilidad.

-----

## Para profundizar

<details>
<summary>Variantes con class-variance-authority (cva)</summary>

Cuando un componente tiene varias dimensiones de variantes (`variant`, `size`, combinaciones entre ellas), un mapa manual se vuelve difícil de mantener. `class-variance-authority` (cva) declara las variantes, sus valores por defecto y las combinaciones en un solo objeto, y devuelve una función que produce el string de clases. shadcn/ui la incluye entre sus dependencias y la usa junto con `cn`. Es una opción, no un requisito: para un botón con dos o tres variantes, el mapa de este ejemplo es suficiente.

</details>

<details>
<summary>Configurar el orden automático de clases</summary>

Instalación y configuración mínima:

```bash
npm install -D prettier prettier-plugin-tailwindcss
```

```json
{
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindStylesheet": "./src/index.css",
  "tailwindFunctions": ["cn", "clsx"]
}
```

* `tailwindStylesheet` es necesaria en Tailwind v4: indica el CSS de entrada para que el plugin conozca tu tema.
* `tailwindFunctions` hace que también ordene las clases dentro de llamadas a esas funciones, no solo en el atributo `className`.

</details>

<details>
<summary>Por qué el orden del atributo no decide el conflicto</summary>

Todas las utilidades de Tailwind tienen la misma especificidad (un selector de clase). Cuando dos reglas empatan en especificidad, la cascada resuelve por **orden de aparición en la hoja de estilos**. Tailwind decide ese orden al generar el CSS, y no depende del orden en que escribes las clases en el HTML. Por eso `class="grid flex"` termina como `display: grid` o `display: flex` según el CSS generado, y no según lo que escribiste primero. `twMerge` evita el problema en el nivel de JavaScript: quita del string la clase que pierde, así que en el DOM queda una sola.

</details>

<details>
<summary>Cuándo usar el modificador importante (`!`)</summary>

Tailwind permite forzar una utilidad con `!` al final (por ejemplo `bg-red-500!`), que la marca como `!important`. Es un último recurso para sobreponerse a CSS heredado que no controlas. Usarlo entre componentes propios suele ser señal de que el conflicto debería resolverse con `cn` o rediseñando el componente. También existe la opción `important` al importar Tailwind, que marca todas las utilidades como `!important` para integrarlas con CSS legado.

</details>

<details>
<summary>Cómo escala el CSS en un proyecto Tailwind</summary>

En CSS tradicional, cada pantalla nueva suele añadir clases nuevas, y el archivo crece con el proyecto. Con utilidades, las mismas clases se reutilizan en cualquier parte, por lo que el CSS generado tiende a estabilizarse a medida que el proyecto crece. El costo se traslada al marcado: cadenas largas, que se controlan con componentes y el orden automático.

</details>

-----

## En entrevista

### Respuesta corta (junior)

Con Tailwind evito la repetición creando componentes: cada componente (un botón, una tarjeta) contiene sus propias clases y se reutiliza importándolo. Para variantes uso un objeto con clases completas y, cuando pueden chocar, una función `cn` que combina `clsx` y `tailwind-merge`. No construyo clases con plantillas de texto, porque Tailwind no las detecta. Uso `@apply` poco, y cuido la accesibilidad con contraste, foco visible y `sr-only`.

### Respuesta ampliada (semi-senior)

* **Utility-first frente a CSS Modules y CSS-in-JS:**
  * *Utility-first (Tailwind):* no hay nombres que inventar, el CSS se genera en build y es estático; el costo es un marcado más largo y la disciplina de extraer componentes.
  * *CSS Modules:* CSS estándar, separado del marcado y aislado por archivo, también sin costo en runtime. Las variantes muy dinámicas son más incómodas y cada componente necesita su propio archivo y nombres.
  * *CSS-in-JS en runtime:* estilos ligados a props y en el mismo archivo, pero con trabajo extra en el navegador y con fricción en Server Components (requiere `'use client'` y configuración de SSR).
* **Escalabilidad:** las utilidades se reutilizan, así que el CSS generado crece más despacio que el CSS escrito a mano. La consistencia depende de los componentes y de los tokens, no de la disciplina de nombres.
* **Design tokens:** con `@theme` (v4) los valores del sistema de diseño viven en un solo lugar y generan sus utilidades (`--color-brand` produce `bg-brand`, `text-brand`, etc.). Usar tokens en lugar de valores arbitrarios (`bg-[#3b82f6]`) evita deriva visual y permite cambiar la marca en un archivo.
* **Rendimiento:** Tailwind genera CSS estático en build, sin costo de generación en el navegador. `cn` sí es JavaScript en runtime (`twMerge` analiza cada string); es un costo pequeño, pero no cero, así que solo se justifica donde hay conflictos posibles.
* **Deuda de mantenimiento:** los riesgos típicos son cadenas duplicadas que se separan con el tiempo, `@apply` usado como CSS paralelo, clases dinámicas que fallan solo en el build, código copiado de versiones distintas de Tailwind y componentes sin accesibilidad. Se mitigan con componentes, `cn`, el plugin de Prettier, revisión de tokens y lint/pruebas de accesibilidad.
* **Componentes de terceros:** copiar componentes (HyperUI, shadcn/ui) da velocidad, pero el código pasa a ser tuyo: hay que adaptarlo a tus tokens y a tu versión.

### Preguntas frecuentes de seguimiento

**1. ¿Cuándo usarías `@apply`?**
Para CSS que no está en un componente propio: sobrescribir estilos de una librería de terceros o dar estilo a HTML que no controlas. En un proyecto con componentes, prefiero extraer un componente.

**2. ¿Para qué sirve `tailwind-merge` si ya tengo `clsx`?**
`clsx` arma el string de clases según condiciones, pero no sabe que `p-2` y `p-4` chocan. `tailwind-merge` detecta el conflicto y conserva la última clase, lo que permite que un `className` externo sobrescriba al del componente.

**3. ¿Por qué `p-2 p-4` no siempre da `p-4`?**
Porque el CSS decide por el orden de las reglas en la hoja generada, no por el orden en el atributo `class`. Con `twMerge` se elimina la clase perdedora antes de renderizar.

**4. ¿Cómo defines variantes de un componente?**
Con una prop tipada como unión de literales y un objeto que mapea cada valor a clases completas. Si las variantes crecen, `class-variance-authority` las declara en un solo lugar.

**5. ¿Cuándo extraes un componente y cuándo no?**
Cuando la duplicación cruza archivos, o cuando el bloque tiene comportamiento y accesibilidad propios. Si la repetición ocurre en un bucle, o dentro de un solo archivo y se corrige con cursores múltiples, no hace falta.

**6. ¿Qué revisas en código de Tailwind generado por IA?**
La versión de Tailwind (`@tailwind` frente a `@import "tailwindcss"`, `bg-gradient-*` frente a `bg-linear-*`, `shadow` renombrado), clases dinámicas, uso de tokens del tema y accesibilidad (contraste, foco visible, etiquetas).

-----

## Siguiente lección

Ya sabes construir componentes con Tailwind, combinar sus variantes y revisarlos con criterio. El paso que sigue es ver esa misma idea llevada a una biblioteca completa de componentes que viven en tu repositorio: [Qué es shadcn/ui](../02-shadcn-ui/01-Qu%C3%A9%20es%20shadcn-ui.md).
