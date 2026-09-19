# Instalación y CLI

## Qué necesitás antes de instalar

shadcn/ui se apoya en Tailwind CSS, así que un proyecto con shadcn/ui es, antes que nada, un proyecto con Tailwind funcionando. Según la documentación actual, es compatible con Next.js, Vite, Laravel, React Router, Astro y TanStack Start, y cada framework tiene su propia guía. Para proyectos nuevos, la documentación recomienda además una herramienta visual (`shadcn/create`) para armar un *preset* y generar el comando de instalación correcto para tu framework.

Acá vemos el caso de **Vite con React y TypeScript**, que es el mismo tipo de proyecto de los ejercicios de práctica, y después Next.js.

> **Sobre el gestor de paquetes:** la documentación de shadcn/ui muestra los comandos con `pnpm` (`pnpm dlx shadcn@latest ...`). Con npm, el equivalente es `npx shadcn@latest ...`. Todos los comandos de esta lección funcionan igual con cualquiera de los dos.

-----

## Instalación en Vite, paso a paso

**1. Crear el proyecto** con la plantilla React + TypeScript:

```bash
pnpm create vite@latest
```

**2. Instalar Tailwind** (como vimos en la lección de integración de Tailwind con Vite):

```bash
pnpm add tailwindcss @tailwindcss/vite
```

Y reemplazar el contenido de `src/index.css` por:

```css
@import "tailwindcss";
```

**3. Configurar el alias `@/`.** Los componentes de shadcn/ui se importan con rutas como `@/components/ui/button`, donde `@` apunta a la carpeta `src`. Para que TypeScript y Vite entiendan ese alias, hay que configurarlo en tres archivos.

En `tsconfig.json` y en `tsconfig.app.json`:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

En `vite.config.ts`, con los tipos de Node instalados (`pnpm add -D @types/node`):

```ts
import path from "path"
import tailwindcss from "@tailwindcss/vite"
import react from "@vitejs/plugin-react"
import { defineConfig } from "vite"

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
})
```

> **Si tu editor marca `baseUrl` como obsoleto** (según la versión de TypeScript), `paths` también funciona sin `baseUrl`, con rutas relativas al `tsconfig` (`"@/*": ["./src/*"]`). La documentación de shadcn/ui muestra la variante con `baseUrl`; si tu proyecto usa una versión reciente de TypeScript y ves una advertencia, esa es la causa más probable.

Los dos lugares hacen trabajos distintos: los `tsconfig` le dicen al **editor y a TypeScript** cómo resolver `@/`, y `vite.config.ts` le dice al **build** cómo resolverlo. Si falta uno, el síntoma típico es que el editor funciona pero el build falla, o al revés.

**4. Inicializar shadcn/ui:**

```bash
pnpm dlx shadcn@latest init
```

El CLI te hace algunas preguntas de configuración (entre ellas, el color base del tema) y genera el archivo `components.json` que vemos en la próxima lección, además de dejar listo el CSS con las variables del tema.

**5. Agregar un componente:**

```bash
pnpm dlx shadcn@latest add button
```

Este comando copia `button.tsx` a `src/components/ui/` e instala las dependencias que ese componente necesita.

**6. Usarlo:**

```tsx
import { Button } from "@/components/ui/button"

function App() {
  return (
    <div className="flex min-h-svh flex-col items-center justify-center">
      <Button>Click me</Button>
    </div>
  )
}

export default App
```

-----

## Instalación en Next.js

En Next.js el proceso es más corto, porque el framework ya viene con el alias `@/` configurado. Con Tailwind ya presente en el proyecto, alcanza con:

```bash
pnpm dlx shadcn@latest init
pnpm dlx shadcn@latest add button
```

`init` configura la integración con Tailwind y el registro de componentes, y `add` copia el componente. El uso es idéntico:

```tsx
import { Button } from "@/components/ui/button"

export default function Home() {
  return (
    <div className="flex min-h-svh items-center justify-center">
      <Button>Click me</Button>
    </div>
  )
}
```

-----

## Los comandos del CLI

`init` y `add` son los que vas a usar el 95% del tiempo, pero el CLI tiene más comandos, que según su referencia oficial son estos:

| Comando | Para qué sirve |
| --- | --- |
| `init` | Inicializa la configuración y las dependencias del proyecto |
| `add` | Agrega componentes al proyecto |
| `view` | Muestra un componente del registro antes de instalarlo |
| `search` | Busca componentes en los registros disponibles |
| `docs` | Trae la documentación de un componente |
| `info` | Muestra la configuración detectada del proyecto |
| `apply` | Aplica un preset (tema, fuente) a un proyecto existente |
| `preset` | Inspecciona los códigos de preset |
| `migrate` | Ejecuta migraciones de código cuando cambia una convención |
| `eject` | Inserta las utilidades de shadcn en tu código y elimina la dependencia |
| `build` | Genera los archivos de un registro propio |

Algunas opciones de `add` que conviene conocer:

```bash
# Agregar varios componentes de una vez
pnpm dlx shadcn@latest add button card dialog

# Ver qué haría, sin escribir nada
pnpm dlx shadcn@latest add button --dry-run
```

Además, `add` acepta `--diff` (para comparar), `--overwrite` (para reemplazar un componente que ya existe), `--all` (para agregar todos) y `--path` (para elegir el destino). `--overwrite` es la que importa cuando querés traer una versión nueva de un componente que ya tenés, y hay que usarla con cuidado, porque **pisa tus modificaciones** (lo vemos en la lección de buenas prácticas).

-----

## Qué queda en tu proyecto después de instalar

Después de `init` y un par de `add`, tu proyecto tiene archivos nuevos que conviene reconocer:

```
mi-proyecto/
├── components.json          ← configuración del CLI
└── src/
    ├── index.css            ← ahora incluye las variables del tema
    ├── components/
    │   └── ui/
    │       └── button.tsx   ← el código del componente, ahora tuyo
    └── lib/
        └── utils.ts         ← expone el helper `cn`
```

Todo eso es código de tu proyecto: va en git, lo revisás en los pull requests, y lo podés editar. Lo que sí vive en `node_modules` son las **dependencias** que los componentes usan (las primitivas, `class-variance-authority`, `lucide-react`), no los componentes en sí.

-----

## Resumen

* shadcn/ui requiere un proyecto con **Tailwind** ya funcionando.
* En **Vite**, hay que configurar el alias `@/` en `tsconfig.json`, `tsconfig.app.json` y `vite.config.ts`; en **Next.js** ya viene configurado.
* El flujo es: `init` una vez (genera `components.json` y el tema), y `add <componente>` cada vez que necesitás uno.
* `add` copia el código a `components/ui/`; ese código es tuyo, y va en git.
* `--dry-run` previsualiza, `--overwrite` reemplaza (pisando tus cambios), `view` y `docs` sirven para informarte antes de instalar.
