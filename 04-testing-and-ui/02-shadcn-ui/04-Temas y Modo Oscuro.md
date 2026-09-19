# Temas y Modo Oscuro

## Cómo se define el aspecto de todo el proyecto

En la lección anterior vimos que los componentes no usan colores fijos: usan tokens como `bg-primary` o `text-muted-foreground`. Esos tokens son **variables CSS**, definidas en un solo lugar, y cambiar su valor cambia el aspecto de toda la aplicación de una vez. Es la razón de que shadcn/ui sea tan fácil de "re-tematizar": no tocás componentes, tocás variables.

-----

## La convención: pares fondo y texto

El sistema de nombres sigue una regla simple. Cada color de **superficie** se nombra sin sufijo, y el color que va **encima** (el texto o los íconos) se nombra igual con `-foreground`:

| Token | Para qué se usa |
| --- | --- |
| `background` / `foreground` | El fondo general de la aplicación y su texto por defecto |
| `card` / `card-foreground` | Superficies elevadas, como tarjetas |
| `primary` / `primary-foreground` | Acciones principales (el botón por defecto) |
| `secondary` / `secondary-foreground` | Acciones de menor énfasis |
| `muted` / `muted-foreground` | Contenido sutil, texto de apoyo |
| `accent` / `accent-foreground` | Estados interactivos, como el elemento resaltado de un menú |
| `destructive` | Acciones destructivas o errores |
| `border`, `input`, `ring` | Bordes, campos de entrada y anillo de foco |
| `radius` | El radio de las esquinas |

La ventaja del par es que **garantiza el contraste**: si cambiás `primary` a un color oscuro, sabés que tenés que ajustar `primary-foreground` a uno claro, y los dos viven juntos, uno al lado del otro, en el CSS. Es la solución al problema de contraste que vimos en la lección de colores de Tailwind.

-----

## Dónde viven: `:root` y `.dark`

Las variables se declaran en tu archivo CSS principal, en dos bloques: uno para el modo claro y otro para el oscuro, con valores en formato **OKLCH** (un formato de color moderno, pensado para que las variaciones de luminosidad se perciban parejas). Los valores del ejemplo son ilustrativos: los tuyos dependen del color base que hayas elegido al inicializar.

```css
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  /* ...el resto de los tokens */
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --primary: oklch(0.922 0 0);
  --primary-foreground: oklch(0.205 0 0);
  /* ...el resto */
}
```

Los valores del bloque `:root` son los del modo claro. Cuando el elemento `<html>` tiene la clase `dark`, las variables del bloque `.dark` pisan a las anteriores, y **los componentes cambian de aspecto sin que cambie una sola clase de Tailwind**. Como los componentes usan `bg-background` y no `bg-white dark:bg-black`, la mayor parte del trabajo del modo oscuro lo hacen las variables, y no hace falta el prefijo `dark:` en cada elemento. Algunos componentes sí lo usan para ajustes puntuales (el código de `Button` de la lección anterior tiene `dark:bg-input/30` en su variante `outline`), pero son refinamientos sobre una base que ya cambia sola.

### Cambiar el color base y las esquinas

El color base (`neutral`, `stone`, `zinc`, `mauve`, `olive`, `mist` o `taupe`) se elige al inicializar el proyecto y determina los valores iniciales de los tokens. Para el radio, hay una sola variable de la que se derivan todas las demás:

```css
--radius-sm: calc(var(--radius) * 0.6);
--radius-md: calc(var(--radius) * 0.8);
--radius-lg: var(--radius);
```

Cambiar `--radius` a `0rem` deja **todas** las esquinas de la aplicación cuadradas; subirlo las redondea todas.

-----

## Agregar un color propio

Si necesitás un color que el tema no trae (por ejemplo, un `warning`), se hace en **dos pasos**: declarar la variable, y exponerla a Tailwind con `@theme inline`, la directiva que vimos en la lección de personalización del tema de Tailwind:

```css
:root {
  --warning: oklch(0.84 0.16 84);
  --warning-foreground: oklch(0.28 0.07 46);
}

@theme inline {
  --color-warning: var(--warning);
  --color-warning-foreground: var(--warning-foreground);
}
```

Con eso, ya existen las clases `bg-warning` y `text-warning-foreground`. Fijate cómo se combinan las dos mecánicas que ya conocés: la variable en `:root` (una variable CSS común, que se puede redefinir en `.dark`), y la referencia dentro de `@theme inline` (que le dice a Tailwind que genere las clases apuntando a esa variable). Por eso hace falta `inline`: la clase tiene que leer la variable **en el momento de usarse**, para que el cambio a modo oscuro se refleje.

Si en el `init` elegís `cssVariables: false` en `components.json`, este mecanismo no existe: los componentes usan clases de Tailwind directas (`bg-zinc-900`), y cambiar el tema es editar componentes. Es una decisión que no se puede revertir sin volver a inicializar, y para la mayoría de los casos el valor por defecto (`true`) es la opción correcta.

-----

## Modo oscuro en Vite

Las variables de `.dark` no hacen nada por sí solas: alguien tiene que agregar la clase `dark` al `<html>` cuando el usuario elige el modo oscuro. La documentación de shadcn/ui para Vite resuelve eso con un **ThemeProvider**, un componente de Context que ya vas a reconocer:

```tsx
import { createContext, useContext, useEffect, useState } from "react"

type Theme = "dark" | "light" | "system"

type ThemeProviderProps = {
  children: React.ReactNode
  defaultTheme?: Theme
  storageKey?: string
}

type ThemeProviderState = {
  theme: Theme
  setTheme: (theme: Theme) => void
}

const initialState: ThemeProviderState = {
  theme: "system",
  setTheme: () => null,
}

const ThemeProviderContext = createContext<ThemeProviderState>(initialState)

export function ThemeProvider({
  children,
  defaultTheme = "system",
  storageKey = "vite-ui-theme",
  ...props
}: ThemeProviderProps) {
  const [theme, setTheme] = useState<Theme>(
    () => (localStorage.getItem(storageKey) as Theme) || defaultTheme
  )

  useEffect(() => {
    const root = window.document.documentElement
    root.classList.remove("light", "dark")

    if (theme === "system") {
      const systemTheme = window.matchMedia("(prefers-color-scheme: dark)")
        .matches
        ? "dark"
        : "light"
      root.classList.add(systemTheme)
      return
    }

    root.classList.add(theme)
  }, [theme])

  const value = {
    theme,
    setTheme: (theme: Theme) => {
      localStorage.setItem(storageKey, theme)
      setTheme(theme)
    },
  }

  return (
    <ThemeProviderContext.Provider {...props} value={value}>
      {children}
    </ThemeProviderContext.Provider>
  )
}

export const useTheme = () => {
  const context = useContext(ThemeProviderContext)
  if (context === undefined)
    throw new Error("useTheme must be used within a ThemeProvider")
  return context
}
```

Se usa envolviendo la aplicación:

```tsx
<ThemeProvider defaultTheme="dark" storageKey="vite-ui-theme">
  <App />
</ThemeProvider>
```

### Leyéndolo con lo que ya sabés

Este componente es una aplicación directa de lo que practicaste, y vale la pena reconocer cada pieza:

* **El estado** vive en el Provider, con `useState`, inicializado **de forma perezosa** desde `localStorage` (la misma técnica del hook `useLocalStorage` del ejercicio de custom hooks).
* **El efecto** sincroniza el estado con el DOM: agrega la clase `light` o `dark` al elemento `<html>` cada vez que `theme` cambia.
* **Tres opciones**, no dos: `"light"`, `"dark"` y `"system"`, que sigue la preferencia del sistema operativo con `prefers-color-scheme`.
* **Persiste** la elección en `localStorage`, así sobrevive a una recarga.
* **Es un Context**: `createContext`, un Provider con `value`, y un hook (`useTheme`) para consumirlo.

Y ahora, un detalle donde el código de la documentación difiere de la receta que armamos en la lección de Context. Fijate cómo se crea el contexto: `createContext<ThemeProviderState>(initialState)`, con un **valor por defecto real**. Eso significa que si alguien usa `useTheme()` fuera de un `ThemeProvider`, `useContext` devuelve ese valor por defecto, no `undefined`, y por lo tanto **la guardia `if (context === undefined)` nunca se dispara**: el error nunca llega, y el `setTheme` por defecto es `() => null`, una función que no hace nada, sin avisar que algo está mal. La receta de nuestra lección de Context (`createContext<T | undefined>(undefined)`) es más estricta, porque hace que el olvido de un Provider sea un error inmediato. Es una decisión de diseño distinta, y vale la pena notarla: es exactamente el tipo de cosa que ves cuando **leés el código en lugar de solo usarlo**.

### El botón para cambiar de tema

Para que el usuario elija, la documentación arma un componente con un menú desplegable:

```tsx
import { Moon, Sun } from "lucide-react"
import { Button } from "@/components/ui/button"
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu"
import { useTheme } from "@/components/theme-provider"

export function ModeToggle() {
  const { setTheme } = useTheme()

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="outline" size="icon">
          <Sun className="h-[1.2rem] w-[1.2rem] scale-100 rotate-0 transition-all dark:scale-0 dark:-rotate-90" />
          <Moon className="absolute h-[1.2rem] w-[1.2rem] scale-0 rotate-90 transition-all dark:scale-100 dark:rotate-0" />
          <span className="sr-only">Toggle theme</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
        <DropdownMenuItem onClick={() => setTheme("light")}>Light</DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme("dark")}>Dark</DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme("system")}>System</DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  )
}
```

Tres cosas para notar. El `<DropdownMenuTrigger asChild>` envuelve un `Button`: usa el mecanismo `asChild` de la lección anterior, para que el disparador del menú **sea** el botón y no un elemento extra alrededor. Los dos íconos (`Sun` y `Moon`) usan el prefijo `dark:` de Tailwind para intercambiarse con una animación: uno se encoge y rota al entrar el modo oscuro, y el otro aparece. Y el `<span className="sr-only">` da un nombre al botón para los lectores de pantalla, ya que el botón solo tiene íconos (la clase `sr-only` de la lección de accesibilidad de Tailwind).

### En Next.js y otros frameworks

La documentación ofrece una guía de modo oscuro para cada framework (Next.js, Vite, Astro, Remix, TanStack Start). En **Next.js**, la guía usa la librería **`next-themes`** en lugar de un ThemeProvider propio, que resuelve además un problema específico del renderizado en el servidor: el parpadeo del tema equivocado antes de que cargue el JavaScript. La idea de fondo es la misma (agregar la clase `dark` al `<html>`); cambia quién lo hace.

-----

## Resumen

* Los componentes usan **tokens semánticos** (`bg-primary`), definidos como variables CSS; cambiar el tema es cambiar las variables, no los componentes.
* Cada superficie tiene un par con `-foreground` para el texto, que garantiza el contraste.
* Las variables se declaran en `:root` (claro) y `.dark` (oscuro), en formato OKLCH; `--radius` controla todas las esquinas.
* Para un color propio: variable en `:root` más una referencia en `@theme inline`, que genera las clases.
* El modo oscuro necesita que alguien agregue la clase `dark` al `<html>`: en Vite, un **ThemeProvider** (un Context con `useState`, `useEffect` y `localStorage`); en Next.js, `next-themes`.
* El ThemeProvider de la documentación crea el contexto con un valor por defecto, por lo que su guardia contra `undefined` nunca actúa: más permisivo que la receta de nuestra lección de Context.
