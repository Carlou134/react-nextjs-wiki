# Anatomía de un Componente

## components.json: la configuración del CLI

Antes de abrir un componente, hay un archivo que explica cómo el CLI decide dónde poner las cosas: `components.json`. Es opcional (solo hace falta si vas a usar el CLI para agregar componentes), y sus opciones principales son estas:

| Opción | Qué controla |
| --- | --- |
| `style` | El estilo visual de los componentes. Actualmente se usa `"new-york"`; el estilo `default` quedó obsoleto. **No se puede cambiar después de `init`.** |
| `tailwind.css` | La ruta del archivo CSS donde se importa Tailwind |
| `tailwind.config` | La ruta del archivo de configuración de Tailwind. **Se deja en blanco con Tailwind v4**, que no lo usa |
| `tailwind.baseColor` | La paleta base del tema (`neutral`, `stone`, `zinc`, `mauve`, `olive`, `mist`, `taupe`). **No se puede cambiar después de `init`.** |
| `tailwind.cssVariables` | Si el tema se genera con variables CSS (`true`, el valor por defecto) o con clases de Tailwind directas (`false`). **No se puede cambiar después de `init`.** |
| `tailwind.prefix` | Un prefijo para las clases de Tailwind de los componentes generados (por ejemplo `tw-`) |
| `rsc` | Si el proyecto usa React Server Components; cuando es `true`, el CLI agrega la directiva `"use client"` a los componentes que la necesitan |
| `tsx` | Si genera componentes en TypeScript (`.tsx`, el valor por defecto) o en JavaScript |
| `aliases` | Las rutas donde el CLI coloca cada cosa: `components`, `ui`, `utils`, `lib`, `hooks` |

Un fragmento típico de los alias:

```json
"aliases": {
  "components": "@/components",
  "ui": "@/components/ui",
  "utils": "@/lib/utils"
}
```

Las tres opciones marcadas como no modificables (`style`, `baseColor`, `cssVariables`) son decisiones que se toman **una vez, al inicializar**. Si te equivocaste, la salida más simple suele ser volver a inicializar el proyecto.

-----

## Abrir el código: el componente Button

Esta es la parte más importante de la lección. El código que el CLI te copia al agregar `button` (tomado del registro oficial, con las cadenas de clases largas acortadas por legibilidad) es este:

```tsx
import * as React from "react"
import { cva, type VariantProps } from "class-variance-authority"
import { cn } from "cn"
import { Slot } from "radix-ui"

const buttonVariants = cva(
  "inline-flex shrink-0 items-center justify-center gap-2 rounded-md text-sm font-medium ...", // clases base
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-white hover:bg-destructive/90 ...",
        outline: "border bg-background shadow-xs hover:bg-accent hover:text-accent-foreground ...",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground ...",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-9 px-4 py-2 has-[>svg]:px-3",
        sm: "h-8 gap-1.5 rounded-md px-3 has-[>svg]:px-2.5",
        lg: "h-10 rounded-md px-6 has-[>svg]:px-4",
        icon: "size-9",
        // ... xs, icon-xs, icon-sm, icon-lg
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

function Button({
  className,
  variant = "default",
  size = "default",
  asChild = false,
  ...props
}: React.ComponentProps<"button"> &
  VariantProps<typeof buttonVariants> & {
    asChild?: boolean
  }) {
  const Comp = asChild ? Slot.Root : "button"

  return (
    <Comp
      data-slot="button"
      data-variant={variant}
      data-size={size}
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  )
}

export { Button, buttonVariants }
```

Son unas cuarenta líneas, y contienen casi todo lo que hay que saber de shadcn/ui, porque todos los componentes siguen la misma forma. Vamos por partes.

-----

## Las piezas, una por una

### 1. Las clases están escritas completas

Fijate en las cadenas de `variants`: `"bg-primary text-primary-foreground hover:bg-primary/90"`. Cada clase aparece **literal**, escrita completa. Esto no es casualidad: es lo que vimos en la lección de Tailwind sobre cómo se detectan las clases. Tailwind lee este archivo como texto y encuentra todas las clases, así que se generan. Si el componente construyera los nombres dinámicamente, se rompería en producción.

### 2. Las clases usan tokens del tema, no colores fijos

`bg-primary`, `text-primary-foreground`, `bg-destructive`, `border-input`: no hay ningún `bg-blue-500`. Los componentes hablan en términos de **tokens semánticos** ("el color primario", "el color destructivo"), que se definen una sola vez en el tema. Es lo que permite cambiar toda la apariencia de la aplicación tocando unas pocas variables (lo vemos en la próxima lección).

### 3. `cva`: las variantes como una API

`cva` (class-variance-authority) es una función que recibe las **clases base** (las que siempre se aplican) y un objeto de **variantes**, y devuelve una función que, dados los valores de esas variantes, calcula la cadena de clases final. Los `defaultVariants` indican qué se usa cuando no se especifica nada.

Por eso el componente se usa así:

```tsx
<Button variant="outline" size="sm">Cancelar</Button>
<Button variant="destructive">Eliminar</Button>
<Button>Guardar</Button>   {/* variant="default" size="default" */}
```

### 4. `VariantProps`: los tipos salen del propio `cva`

```tsx
React.ComponentProps<"button"> & VariantProps<typeof buttonVariants> & { asChild?: boolean }
```

Los tipos de las props se construyen combinando tres cosas. `React.ComponentProps<"button">` incluye **todas** las props que acepta un `<button>` nativo (`onClick`, `disabled`, `type`...), así el componente se comporta como un botón común. `VariantProps<typeof buttonVariants>` **deriva automáticamente** los valores válidos de `variant` y `size` a partir de la definición de `cva`, de modo que si agregás una variante nueva, el tipo se actualiza solo. Y `{ asChild?: boolean }` agrega la prop propia. Es el patrón de tipado que vimos al hablar de props, aplicado a un componente real.

> **Sobre `ref`:** el componente no usa `forwardRef`. En React 19 `ref` es una prop común, y ya viene incluida en `React.ComponentProps<"button">`. Es la simplificación de React 19 que vimos en su lección, aplicada en el código de shadcn/ui.

### 5. `cn`: combinar clases sin conflictos

```tsx
className={cn(buttonVariants({ variant, size, className }))}
```

`cn` es el helper que combina cadenas de clases. Fijate que la prop `className` que recibe el componente se pasa **adentro** de `buttonVariants`, así que va **al final** de la cadena. Esto es lo que permite personalizar un botón puntual desde afuera:

```tsx
<Button className="w-full">Ocupa todo el ancho</Button>
```

Y `cn`, además de unir las clases, se ocupa de **resolver los conflictos** entre ellas (como vimos en la lección de buenas prácticas de Tailwind: si el botón trae `px-4` y vos pasás `px-8`, tiene que ganar la tuya). Históricamente `cn` se definía en `lib/utils.ts` combinando `clsx` y `tailwind-merge`; según el changelog oficial, desde septiembre de 2026 vive en un paquete propio, y el código que genera el CLI lo importa desde `"cn"`. **Abrí el `lib/utils.ts` de tu proyecto** para ver cuál de las dos formas tiene tu versión: puede variar según cuándo lo hayas instalado, y el comando `shadcn migrate cn` existe para actualizar proyectos anteriores.

### 6. `asChild` y `Slot`: cambiar la etiqueta sin perder el estilo

```tsx
const Comp = asChild ? Slot.Root : "button"
```

Por defecto, `Button` renderiza un `<button>`. Pero a veces necesitás que **se vea como un botón y sea otra cosa**: el caso típico es un enlace. Con `asChild`, en lugar de renderizar su propio `<button>`, el componente le **pasa sus props y estilos al hijo**:

```tsx
<Button asChild>
  <a href="/login">Iniciar sesión</a>
</Button>
```

El resultado es una etiqueta `<a>` (correcta semánticamente: es un enlace), con el aspecto de un botón. Es el mismo patrón que vas a ver en casi todos los componentes de shadcn/ui, por ejemplo `<DialogTrigger asChild>`.

> **Radix y Base UI:** `asChild` es el mecanismo de la variante construida sobre **Radix UI**, que es la que mostramos en este módulo. La variante sobre **Base UI** (la opción por defecto desde julio de 2026, según el changelog) resuelve lo mismo con una prop `render`, y su documentación advierte específicamente sobre cómo usarla con enlaces. Los conceptos (variantes, tokens, `cn`, composición) son idénticos; cambia el detalle de esta prop. Si el código que te genera el CLI usa `render` en lugar de `asChild`, no es un error: es la otra base.

### 7. `data-slot`: identificar cada parte

```tsx
data-slot="button"
data-variant={variant}
data-size={size}
```

Son atributos de datos que identifican la parte del componente y su estado actual. Sirven para poder **apuntar a esa parte desde afuera** con CSS o con variantes de Tailwind basadas en atributos, sin depender de nombres de clase internos. Vas a verlos repetidos en todos los componentes.

-----

## Personalizar: editá el archivo

Como el código es tuyo, agregar una variante es editar el objeto de `cva`. Por ejemplo, un botón de éxito:

```tsx
variants: {
  variant: {
    default: "bg-primary text-primary-foreground hover:bg-primary/90",
    // ...las demás
    success: "bg-green-600 text-white hover:bg-green-700",
  },
```

Y sin tocar nada más, el tipo de `variant` ya incluye `"success"` (por `VariantProps`), y podés usar `<Button variant="success">`. Esta es la ventaja concreta de que el código esté en tu proyecto: no hay ninguna API de "temas" ni extensión que aprender, es editar un objeto.

Hay dos formas de personalizar, y conviene elegir con criterio:

* **Editar el componente base** (`button.tsx`), cuando el cambio tiene que aplicar a **todos** los botones de la aplicación: una variante nueva, un ajuste de tamaño.
* **Envolverlo en un componente propio**, cuando el cambio es específico de un caso: un `SubmitButton` que siempre muestra un indicador de carga, o un `DangerButton` con confirmación. El original queda intacto, y tu componente compone sobre él.

-----

## Resumen

* `components.json` le dice al CLI **dónde** poner cada cosa y con qué opciones; `style`, `baseColor` y `cssVariables` se deciden una vez, en `init`.
* Un componente de shadcn/ui es un archivo corto que combina: **clases de Tailwind completas y literales**, **tokens del tema** (`bg-primary`), **`cva`** para las variantes, **`VariantProps`** para los tipos, y **`cn`** para combinar clases.
* La prop `className` se pasa al final de `cn`, así que siempre podés sobrescribir un estilo puntual desde afuera.
* `asChild` (Radix) o `render` (Base UI) permite renderizar otra etiqueta conservando el estilo, como un enlace con aspecto de botón.
* Como el código es tuyo, personalizar es **editar el archivo** (para cambios globales) o **envolver el componente** (para casos puntuales).
