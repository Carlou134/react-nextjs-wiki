# Cómo Tailwind detecta las clases

## En una frase

Tailwind **no ejecuta tu código**: escanea tus archivos como texto plano, busca cadenas que coincidan con clases existentes y solo genera el CSS de las que encuentra escritas **completas**.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es una clase de utilidad y cómo se aplica en el marcado: [Fundamentos de Tailwind](01-Fundamentos%20de%20Tailwind.md).
* Cómo se instala Tailwind v4 y dónde se importa (`@import "tailwindcss"`): [Instalación e Integración](02-Instalaci%C3%B3n%20e%20Integraci%C3%B3n.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Escaneo de código fuente:** lectura de tus archivos como texto para descubrir qué clases se usan.
* **Candidato:** token del texto que *podría* ser una clase. Tailwind lo compara con las utilidades que conoce; si no coincide, lo descarta.
* **Clase dinámica:** nombre de clase armado en tiempo de ejecución, por concatenación o interpolación (`` `bg-${color}-600` ``).
* **Mapa de clases:** objeto o tabla donde cada opción guarda su clase escrita completa.
* **Safelist:** lista de clases que se deben generar aunque no aparezcan en ningún archivo. En v4 se declara con `@source inline()`.

-----

## El problema

Este componente parece correcto:

```tsx
function Button({ color, children }) {
  return (
    <button className={`bg-${color}-600 hover:bg-${color}-500`}>
      {children}
    </button>
  );
}
```

Cuando `color` vale `"blue"`, en tiempo de ejecución el resultado es `bg-blue-600`. Pero Tailwind trabaja **antes**, al compilar el CSS, y nunca evalúa esa plantilla. Lo que ve en el archivo es el texto `bg-${color}-600`, que no es una clase válida.

Consecuencia: `bg-blue-600` no se genera, la regla no existe en el CSS final y el botón se ve sin estilo.

El error es traicionero porque a veces no falla de inmediato. Si la clase completa aparece literalmente en otro archivo, Tailwind la genera por esa otra ocurrencia y el componente parece funcionar. Cuando esa otra ocurrencia desaparece, el estilo se rompe sin que el componente haya cambiado.

-----

## Cómo funciona

### El proceso de detección

Tailwind no parsea JavaScript, JSX ni HTML como código. Trata cada archivo como una cadena de texto, extrae los tokens que tienen forma de nombre de clase y genera CSS solo para los que corresponden a una utilidad conocida.

```
  Archivos del proyecto (.tsx, .html, .vue, .md ...)
            |
            |  1. Escaneo como TEXTO PLANO (no se ejecuta nada)
            v
  Candidatos:  "bg-blue-600"   "flex"   "hover:bg-blue-500"
               "bg-${color}-600"   "function"   "return" ...
            |
            |  2. Comparación con las utilidades que Tailwind conoce
            |     (coincide -> se conserva; no coincide -> se descarta)
            v
  Clases válidas:  bg-blue-600   flex   hover:bg-blue-500
            |
            |  3. Generación
            v
  CSS final:  solo las reglas de esas clases
```

Del paso 2 se desprende algo importante: Tailwind no distingue una clase usada de una clase mencionada. Un token válido dentro de un comentario, una cadena o un archivo Markdown también genera CSS. Y un token incompleto como `bg-${color}-600` nunca coincide con nada.

### Qué archivos se escanean (v4)

En Tailwind v4 la detección es **automática**: no hay que indicar dónde buscar. Se escanean todos los archivos del proyecto, excepto:

* los listados en `.gitignore`,
* la carpeta `node_modules`,
* los archivos binarios (imágenes, videos, zip),
* los archivos CSS,
* los archivos de bloqueo de gestores de paquetes (`package-lock.json`, `pnpm-lock.yaml`, etc.).

Dos consecuencias prácticas:

* Si una clase vive **solo** en un archivo ignorado por git, no se detecta.
* Las clases usadas dentro de una librería instalada en `node_modules` (por ejemplo, una librería de UI hecha con Tailwind) **no se detectan por defecto**.

> **v3 frente a v4.** En v3 había que declarar las rutas en la opción `content` de `tailwind.config.js`, y las clases forzadas se declaraban en `safelist`. En v4 la detección es automática, las rutas extra se declaran en CSS con `@source` y la safelist se reemplaza por `@source inline()`. Según la guía oficial de migración, `safelist` no se admite en la configuración JavaScript de v4.

### Clases completas: mapas y condicionales

La regla es que **cada clase que necesites debe aparecer completa, como texto, en algún archivo**. Para valores dinámicos, la solución estándar es un mapa de clases:

```tsx
const colorVariants = {
  blue: "bg-blue-600 hover:bg-blue-500",
  red: "bg-red-600 hover:bg-red-500",
};

function Button({ color, children }) {
  return <button className={colorVariants[color]}>{children}</button>;
}
```

Al escanear, Tailwind ve las cadenas completas `"bg-blue-600 hover:bg-blue-500"` y `"bg-red-600 hover:bg-red-500"` y genera todas esas clases. En ejecución, `colorVariants[color]` solo **elige** una de las cadenas que ya existen.

Un condicional con clases literales funciona por la misma razón:

```tsx
<div className={isActive ? "bg-blue-500" : "bg-gray-500"} />
```

Ambas clases están escritas completas; qué clase se aplica se decide en ejecución, pero las dos ya fueron detectadas.

Lo que se debe evitar es **armar el nombre**, no elegir entre nombres completos:

```js
"bg-" + color        // Tailwind ve: "bg-" + color
`text-${size}`       // Tailwind ve: text-${size}
`w-[${px}px]`        // también falla: el valor arbitrario es dinámico
```

### Rutas adicionales con `@source`

La directiva `@source` agrega rutas al escaneo. Las rutas son relativas al archivo CSS donde se declaran:

```css
@import "tailwindcss";
@source "../node_modules/@acmecorp/ui-lib";
```

Otras formas, todas en CSS:

```css
/* Excluir una ruta del escaneo */
@source not "../src/components/legacy";

/* Cambiar la ruta base del escaneo (útil en monorepos) */
@import "tailwindcss" source("../src");

/* Desactivar la detección automática y declarar solo las rutas necesarias */
@import "tailwindcss" source(none);
@source "../admin";
@source "../shared";
```

`source(none)` sirve, por ejemplo, cuando hay varias hojas de estilo Tailwind con conjuntos de clases aislados.

### Forzar clases con `@source inline()`

A veces una clase **no está escrita en ningún archivo**. Es el caso típico de nombres que llegan desde una base de datos o un CMS. `@source inline()` le indica a Tailwind que genere esas clases sin buscarlas en el código:

```css
@import "tailwindcss";
@source inline("underline");
```

Acepta expansión de llaves para generar variantes y rangos sin escribir cada clase:

```css
@source inline("{hover:,focus:,}underline");
@source inline("{hover:,}bg-red-{50,{100..900..100},950}");
```

La segunda línea genera `bg-red-50`, `bg-red-100`, `bg-red-200`, hasta `bg-red-950`, y también sus versiones con `hover:`.

Para el caso contrario existe `@source not inline()`, que evita que se generen ciertas clases:

```css
@source not inline("{hover:,focus:,}bg-red-{50,{100..900..100},950}");
```

-----

## Ejemplo completo

Un botón con variantes de color y tamaño, con clases completas y tipos derivados del propio mapa:

```tsx
const colorVariants = {
  blue: "bg-blue-600 hover:bg-blue-500 text-white",
  red: "bg-red-600 hover:bg-red-500 text-white",
  yellow: "bg-yellow-300 hover:bg-yellow-400 text-black",
} as const;

const sizeVariants = {
  sm: "px-2 py-1 text-sm",
  md: "px-4 py-2 text-base",
} as const;

type Color = keyof typeof colorVariants; // "blue" | "red" | "yellow"
type Size = keyof typeof sizeVariants;   // "sm" | "md"

type ButtonProps = {
  color: Color;
  size?: Size;
  children: React.ReactNode;
};

export function Button({ color, size = "md", children }: ButtonProps) {
  return (
    <button
      className={`${colorVariants[color]} ${sizeVariants[size]} rounded font-medium`}
    >
      {children}
    </button>
  );
}
```

Puntos clave:

1. Cada clase aparece completa en un literal, así que Tailwind la detecta.
2. La interpolación `${colorVariants[color]}` es válida porque solo **inserta** una cadena que ya fue escaneada; no construye el nombre de una clase.
3. `as const` y `keyof typeof` derivan las opciones válidas del propio objeto. Agregar un color es agregar una línea, y TypeScript avisa si se pasa un color inexistente.

-----

## En React

* Los mapas de clases se ubican fuera del componente, como constantes del módulo. Así no se recrean en cada render.
* El mismo principio aplica a utilidades como `clsx` o `cva`: funcionan porque reciben cadenas completas. Si se les pasa un nombre armado por concatenación, fallan igual.
* En Next.js y otros entornos con Vite o PostCSS, el escaneo ocurre en tiempo de compilación, sin importar si el componente es de servidor o de cliente: solo importa el texto del archivo.

-----

## Errores comunes

### 1. Interpolar parte del nombre de la clase

```tsx
<div className={`text-${size}`} />
```

**Qué pasa:** la clase no existe en el CSS y el elemento no tiene el estilo.
**Por qué:** Tailwind ve el texto `text-${size}`, que no coincide con ninguna utilidad.
**Arreglo:** mapa de clases con cada opción completa (`{ sm: "text-sm", lg: "text-lg" }`).

### 2. Depender de que otra ocurrencia genere la clase

**Qué pasa:** el componente funciona en un momento y se rompe después de un cambio en otro archivo.
**Por qué:** la clase solo se generaba porque aparecía completa en otro lugar del proyecto.
**Arreglo:** escribir cada clase completa en el archivo que la usa.

### 3. Esperar que se detecten clases de una librería en `node_modules`

**Qué pasa:** los componentes de una librería de UI basada en Tailwind aparecen sin estilos.
**Por qué:** `node_modules` está excluido del escaneo automático.
**Arreglo:** agregar `@source "../node_modules/nombre-de-la-libreria";` en el CSS.

### 4. Clases solo en archivos ignorados por git

**Qué pasa:** una clase presente en un archivo generado o listado en `.gitignore` no produce CSS.
**Por qué:** el escaneo automático respeta `.gitignore`.
**Arreglo:** registrar la ruta con `@source`, o mover la clase a un archivo versionado.

### 5. Usar `safelist` o `content` de v3 en un proyecto v4

**Qué pasa:** la configuración se ignora o no tiene efecto.
**Por qué:** en v4 la detección es automática y `safelist` no se admite en la configuración JavaScript.
**Arreglo:** usar `@source` para rutas y `@source inline()` para clases forzadas.

-----

## Cuándo sí y cuándo no

**Usa mapas de clases cuando** una prop o un estado determina el estilo (variantes de color, tamaños, estados).

**Usa `@source`** cuando las clases viven en una ubicación que el escaneo automático no cubre: librerías en `node_modules`, carpetas ignoradas o proyectos con varias raíces.

**Usa `@source inline()` solo cuando** el nombre de la clase no existe en el código (por ejemplo, viene de un CMS). Es un último recurso: si puedes resolver el caso con un mapa, hazlo, porque deja las clases visibles y buscables en el código.

**No uses** concatenación ni interpolación para construir nombres de clase, ni siquiera "solo esta vez".

-----

## Resumen en 5 líneas

1. Tailwind escanea los archivos como **texto plano** y genera CSS solo para los tokens que coinciden con una utilidad conocida.
2. Las clases armadas dinámicamente (`` `bg-${color}-600` ``) **no se generan**: cada clase debe estar escrita completa.
3. La solución estándar es un **mapa de clases** o un condicional con clases literales.
4. En v4 la detección es automática, pero excluye `.gitignore`, `node_modules`, binarios, CSS y archivos de bloqueo.
5. `@source` agrega o excluye rutas, y `@source inline()` fuerza clases que no aparecen en ningún archivo.

-----

## Para profundizar

<details>
<summary>Tailwind escanea texto, no es un análisis de uso</summary>

El escaneo no entiende el contexto. No sabe si un token está en un componente, un comentario o una cadena de documentación. Si el token coincide con una utilidad, se genera. Por eso una clase mencionada en un comentario o en un archivo Markdown del proyecto puede terminar en el CSS aunque ningún elemento la use. La contrapartida es la garantía inversa: nada que no exista como texto completo se genera jamás.

</details>

<details>
<summary>Cambiar la ruta base con `source()`</summary>

Por defecto, la ruta base del escaneo es el directorio de trabajo desde donde se ejecuta la compilación. En un monorepo, si el comando corre desde la raíz y las clases están en un paquete, se puede fijar otra base:

```css
@import "tailwindcss" source("../src");
```

Las rutas de `@source` se resuelven relativas al archivo CSS donde se declaran, no al directorio de trabajo.

</details>

<details>
<summary>Bibliotecas de variantes (`cva`, `tailwind-variants`)</summary>

Estas bibliotecas evitan escribir mapas a mano, pero siguen la misma regla: reciben cadenas completas y las combinan en ejecución. Como Tailwind escanea el archivo donde se declaran esas cadenas, las detecta sin configuración adicional. No cambian el mecanismo de detección; solo lo organizan.

</details>

-----

## En entrevista

### Respuesta corta (junior)

Tailwind no ejecuta el código: lee los archivos como texto y genera CSS solo para las clases que encuentra escritas completas. Por eso no se pueden armar nombres como `` `bg-${color}-600` ``. Se usa un mapa de clases con cada opción escrita completa.

### Respuesta ampliada (semi-senior)

* **Mecanismo:** Tailwind extrae candidatos del texto plano y los compara con las utilidades que conoce. Los que no coinciden se descartan. No hay análisis de sintaxis ni ejecución.
* **Clases dinámicas:** la concatenación o interpolación de partes del nombre produce cadenas que nunca coinciden. Se debe elegir entre cadenas completas mediante mapas o condicionales.
* **Fallo silencioso:** puede parecer que funciona si la clase completa aparece en otro archivo, y romperse después. No hay error de compilación.
* **Detección en v4:** automática sobre todo el proyecto, excepto `.gitignore`, `node_modules`, binarios, CSS y archivos de bloqueo.
* **Ajustes:** `@source` agrega rutas, `@source not` excluye, `source(none)` desactiva la detección automática y `source("ruta")` cambia la base.
* **Forzar clases:** `@source inline()` con expansión de llaves reemplaza a la `safelist` de v3; `@source not inline()` evita generar clases.
* **v3 frente a v4:** en v3 se usaba `content` y `safelist` en `tailwind.config.js`; en v4 se usa CSS. `safelist` no se admite en la configuración JavaScript de v4.
* **Tipado:** con `as const` y `keyof typeof` el mapa de clases define el tipo de las variantes, y se evitan colores inválidos en compilación.

### Preguntas frecuentes de seguimiento

**1. ¿Por qué `bg-${color}-600` no funciona?**
Porque Tailwind no evalúa la plantilla: ve el texto `bg-${color}-600`, que no coincide con ninguna utilidad. Ninguna clase completa se genera.

**2. ¿Por qué `` `${colors[color]} rounded` `` sí funciona?**
Porque la interpolación solo inserta una cadena que ya existe completa en el objeto `colors`, y esa cadena fue detectada al escanear el archivo. Lo que rompe la detección es construir el nombre de la clase, no combinar cadenas completas.

**3. ¿Un condicional `isActive ? "bg-blue-500" : "bg-gray-500"` es seguro?**
Sí. Ambas clases están escritas completas, así que se generan las dos. La decisión de cuál aplicar ocurre en ejecución.

**4. ¿Cómo hago que Tailwind detecte clases de una librería en `node_modules`?**
Con `@source` apuntando a la librería, por ejemplo `@source "../node_modules/@acmecorp/ui-lib";`. El escaneo automático excluye `node_modules`.

**5. ¿Cómo reemplazo la `safelist` de v3 en v4?**
Con `@source inline()`, que admite expansión de llaves: `@source inline("{hover:,}bg-red-{50,{100..900..100},950}")`. La `safelist` de la configuración JavaScript no se admite en v4.

**6. ¿Cuándo conviene `@source inline()` frente a un mapa de clases?**
Solo cuando el nombre de la clase no está en el código, por ejemplo si viene de un CMS o de la base de datos. Si las opciones se conocen de antemano, un mapa es mejor: las clases quedan visibles y se pueden buscar en el repositorio.

-----

## Siguiente lección

Ahora que sabes cómo Tailwind decide qué CSS generar, el paso que sigue es usar las utilidades más comunes para dar forma al contenido: [Tipografía, Espaciado y Colores](04-Tipograf%C3%ADa%2C%20Espaciado%20y%20Colores.md).
