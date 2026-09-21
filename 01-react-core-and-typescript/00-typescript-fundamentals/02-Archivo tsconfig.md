# El archivo tsconfig.json

## En una frase

`tsconfig.json` es el archivo donde se declara **qué archivos forman el proyecto TypeScript y con qué reglas se revisan y se traducen**. Lo leen el compilador (`tsc`), el editor y muchas herramientas.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es TypeScript y cómo se anotan los tipos: [Types](01-Types.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Compilador (`tsc`):** programa de TypeScript que revisa los tipos y, si se pide, genera JavaScript.
* **Transpilar:** traducir código de un lenguaje a otro del mismo nivel (aquí, de TypeScript a JavaScript).
* **Bundler:** herramienta (Vite, webpack, esbuild) que junta los módulos de una app en archivos listos para el navegador.
* **Resolución de módulos:** las reglas con las que se decide qué archivo corresponde a un `import`.
* **JSONC:** JSON que admite comentarios y comas finales. `tsconfig.json` funciona así.

-----

## El problema

Sin configuración, `tsc archivo.ts` usaría valores por defecto y habría que repetir opciones por consola:

```bash
tsc src/main.ts --target ES2022 --strict --noEmit
```

Esto no escala: cada persona del equipo y cada herramienta (editor, CI, bundler) necesitaría las mismas opciones, y bastaría una diferencia para que el mismo código dé errores en un lugar y no en otro.

`tsconfig.json` guarda esas decisiones en un solo archivo versionado. Con él basta ejecutar `tsc` sin argumentos, y el editor aplica las mismas reglas mientras escribes.

-----

## Cómo funciona

### Estructura básica

El archivo tiene tres piezas principales:

```json
{
  "compilerOptions": { },
  "include": [],
  "exclude": []
}
```

* `compilerOptions`: las reglas del compilador (estrictez, versión de JavaScript, módulos, JSX).
* `include`: patrones (globs) de los archivos que forman el proyecto. Si se omite junto con `files`, se toma todo lo que haya bajo la carpeta del `tsconfig.json`.
* `exclude`: patrones que se quitan de lo que dejó `include`. Si no lo defines, por defecto excluye `node_modules`, `bower_components`, `jspm_packages` y el `outDir`. Ojo: `exclude` solo filtra `include`; un archivo excluido igualmente entra al proyecto si otro archivo lo importa.

Existe también `files`, una lista explícita de archivos (sin patrones). Se usa poco, salvo en el caso de las referencias que se ve más abajo.

Para generar un archivo inicial (desde TypeScript 5.9 es una configuración breve con valores recomendados, no una lista larga de opciones comentadas):

```bash
npx tsc --init
```

### Las opciones que más aparecen

**`strict`**

Es un interruptor que activa a la vez varias comprobaciones estrictas: `strictNullChecks` (`null` y `undefined` no son valores válidos de cualquier tipo), `noImplicitAny` (prohíbe que un valor quede como `any` por omisión), `strictFunctionTypes`, `strictPropertyInitialization`, `useUnknownInCatchVariables`, `noImplicitThis`, `alwaysStrict`, entre otras. Desde TypeScript 6.0 su valor por defecto es `true` (antes era `false`), pero conviene declararlo de forma explícita. Se puede desactivar una en particular después: `"strict": true, "noImplicitAny": false`.

**`target`**

Versión de JavaScript que se **emite**. Con `"target": "ES2022"`, la sintaxis moderna se conserva; con un target más bajo, TypeScript la reescribe (por ejemplo, convierte `async/await` en código equivalente). También cambia el valor por defecto de `lib`. En una app con bundler, el target suele fijarse según los navegadores que se quieren soportar.

**`lib`**

Qué **declaraciones de tipos** del entorno conoce el compilador: `ES2022` (los métodos del lenguaje, como `Array.prototype.at`), `DOM` (`document`, `window`, `fetch`), `DOM.Iterable`. No agrega código al resultado: solo dice qué APIs existen. Si usas `document` y `lib` no incluye `DOM`, TypeScript marcará error.

**`module` y `moduleResolution`**

* `module`: el formato de módulos que se emite (`ESNext`, `CommonJS`, `NodeNext`).
* `moduleResolution`: cómo se busca el archivo de cada `import`. Con un bundler moderno se usa `"bundler"` (disponible desde TypeScript 5.0): permite imports sin extensión, como hacen Vite y webpack. Para Node.js puro se usa `"NodeNext"`, que exige extensiones explícitas en los imports relativos de módulos ES.

**`jsx`**

Cómo se trata el JSX. `"react-jsx"` usa la transformación automática de React 17+: no hace falta `import React` en cada archivo. Otros valores: `"preserve"` (deja el JSX intacto para que lo procese otra herramienta; lo usa Next.js) y `"react"` (transformación clásica).

**`noEmit`**

Con `true`, `tsc` solo **revisa tipos** y no escribe archivos `.js`. Es lo habitual en proyectos con Vite o Next.js, donde el bundler (no `tsc`) genera el JavaScript. Así `tsc` queda como verificador de tipos.

**`noUnusedLocals` y `noUnusedParameters`**

Marcan como error las variables locales y los parámetros que nunca se usan. Ayudan a mantener limpio el código. Un parámetro que se deja sin usar a propósito se marca con un prefijo `_` (`_event`) para que no se reporte.

**`skipLibCheck`**

Omite la revisión de los archivos de declaración (`.d.ts`), incluidos los de `node_modules`. Acelera la compilación y evita errores por conflictos entre librerías ajenas. No afecta la revisión de **tu** código.

**`paths`**

Define alias de importación, por ejemplo `"@/*": ["./src/*"]` para escribir `import { Button } from "@/components/Button"`. Importante: `paths` solo le enseña la ruta a **TypeScript**. El bundler debe configurar el mismo alias por su cuenta (en Vite, `resolve.alias`; Next.js lo lee del `tsconfig.json`).

### Proyectos Vite: varios archivos con referencias

Una plantilla moderna de Vite con React y TypeScript trae tres archivos:

| Archivo | Rol |
|---|---|
| `tsconfig.json` | Raíz. No revisa nada por sí mismo: solo apunta a los demás con `references`. |
| `tsconfig.app.json` | Configuración del código de la aplicación (`src`), que corre en el navegador. |
| `tsconfig.node.json` | Configuración de los archivos que corren en Node, como `vite.config.ts`. |

El archivo raíz suele verse así:

```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ]
}
```

Motivo de la separación: el código de la app necesita tipos del navegador (`DOM`), mientras que `vite.config.ts` necesita tipos de Node y no debe ver `document`. Con un solo archivo habría que mezclar ambos entornos. Con `"files": []`, la raíz no incluye ningún archivo propio, y cada sub-proyecto se revisa con sus reglas. Los scripts de la plantilla ejecutan `tsc -b` (modo *build*), que recorre las referencias.

Además, las reglas que te interesan (como `strict` o `paths`) se editan en `tsconfig.app.json`, no en la raíz.

-----

## Ejemplo completo

Un `tsconfig.app.json` representativo de un proyecto Vite con React. Los valores exactos cambian entre versiones de la plantilla, pero la estructura es esta:

```jsonc
{
  "compilerOptions": {
    // Qué versión de JavaScript se emite y qué APIs conoce el compilador
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],

    // Módulos: formato ESM y resolución al estilo de un bundler
    "module": "ESNext",
    "moduleResolution": "bundler",

    // JSX con la transformación automática de React 17+
    "jsx": "react-jsx",

    // Solo verificar tipos; Vite genera el JavaScript
    "noEmit": true,

    // Revisión estricta (incluye strictNullChecks, noImplicitAny, etc.)
    "strict": true,

    // Higiene del código
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,

    // No revisar los .d.ts de node_modules
    "skipLibCheck": true,

    // Alias opcional: import { X } from "@/components/X"
    // (Vite necesita además resolve.alias con el mismo alias)
    "paths": { "@/*": ["./src/*"] }
  },

  // Solo el código de la aplicación
  "include": ["src"]
}
```

-----

## Errores comunes

### 1. Cambiar `tsconfig.json` y no ver efecto

**Qué pasa:** en un proyecto Vite, activar `strict` o `paths` en la raíz no hace nada.
**Por qué:** la raíz tiene `"files": []` y solo referencia a los demás. Las reglas viven en los archivos referenciados.
**Cómo se arregla:** edita `tsconfig.app.json` (código de la app) o `tsconfig.node.json` (configuración de Vite).

### 2. El alias `@/` funciona en el editor pero falla al ejecutar

**Qué pasa:** el editor no marca error, pero Vite dice que no encuentra el módulo `@/components/...`.
**Por qué:** `paths` es solo información para TypeScript; no cambia cómo el bundler resuelve los imports.
**Cómo se arregla:** declara el mismo alias en el bundler (en Vite, `resolve.alias` en `vite.config.ts`).

### 3. Error "Cannot find name 'document'" o "Cannot find name 'console'"

**Qué pasa:** el compilador no reconoce APIs del navegador o de Node.
**Por qué:** `lib` no incluye las declaraciones necesarias (por ejemplo, falta `DOM`), o el archivo pertenece al `tsconfig` equivocado.
**Cómo se arregla:** añade `"DOM"` a `lib` para código de navegador, o revisa que el archivo esté cubierto por el `include` del `tsconfig` adecuado.

### 4. Un archivo excluido sigue apareciendo en la compilación

**Qué pasa:** aparece en la salida de `tsc --listFiles` un archivo que estaba en `exclude`.
**Por qué:** `exclude` solo filtra el resultado de `include`. Si otro archivo del proyecto lo importa, entra igualmente.
**Cómo se arregla:** quita el `import`, o acepta que `exclude` no significa "TypeScript nunca lo lee".

### 5. Errores en `node_modules` que no son tuyos

**Qué pasa:** el compilador reporta errores dentro de archivos `.d.ts` de una librería.
**Por qué:** dos librerías declaran tipos incompatibles, o la librería usa una versión de TypeScript más nueva.
**Cómo se arregla:** activa `"skipLibCheck": true`. Es lo habitual, con el costo de no detectar errores en esos `.d.ts`.

### 6. Quedarse con `strict: false` para "avanzar más rápido"

**Qué pasa:** el código compila con muchos `any` implícitos y accesos a `undefined` sin revisar.
**Por qué:** sin `strict` desaparece buena parte de lo que TypeScript aporta; los errores aparecen en ejecución.
**Cómo se arregla:** empieza con `strict: true`. En una migración de un proyecto JavaScript grande, se puede activar por partes (una opción a la vez) en lugar de dejarlo apagado.

-----

## Cuándo sí y cuándo no

**Ajusta el `tsconfig` cuando:**

* Necesitas un alias de importación (`paths`).
* Quieres endurecer o relajar reglas concretas (`noUnusedLocals`, `noImplicitAny`).
* Agregas código para otro entorno (scripts de Node, pruebas) que requiere otros tipos.

**No lo toques si:**

* La plantilla del framework (Vite, Next.js) ya lo configuró: sus valores están pensados para su bundler. Cambia una opción solo si sabes qué efecto tiene.
* El problema es un error de tipos en tu código: se arregla en el código, no relajando el compilador.

-----

## Resumen en 5 líneas

1. `tsconfig.json` define qué archivos entran al proyecto (`include`/`exclude`) y con qué reglas se revisan (`compilerOptions`).
2. `strict: true` activa un conjunto de comprobaciones estrictas y es la base recomendada.
3. `target` es qué JavaScript se emite; `lib` es qué APIs conoce el compilador; `module`/`moduleResolution` controlan los imports.
4. En proyectos con bundler, `noEmit: true` deja a `tsc` solo como verificador; `paths` requiere el mismo alias en el bundler.
5. Vite usa un `tsconfig.json` raíz con referencias a `tsconfig.app.json` y `tsconfig.node.json`; las reglas se editan en estos dos.

-----

## Para profundizar

<details>
<summary>Qué hace `tsc -b` y por qué las referencias importan</summary>

`tsc -b` (*build mode*) compila los proyectos referenciados en el orden correcto y puede evitar recompilar los que no cambiaron. Según la documentación oficial, un proyecto referenciado debe tener `composite` habilitado (que a su vez exige `declaration`). Las plantillas de Vite traen además opciones propias, como `tsBuildInfoFile`, que cambian entre versiones. Para editar una plantilla existente no necesitas conocer esos detalles, pero explican por qué los archivos `tsconfig.app.json` y `tsconfig.node.json` traen más opciones que una configuración simple.

</details>

<details>
<summary>extends: compartir configuración entre archivos</summary>

Un `tsconfig` puede heredar de otro con `"extends": "./tsconfig.base.json"`. Las opciones del archivo hijo sobrescriben las del padre, y `include`/`exclude`/`files` del hijo reemplazan a los del padre. Es útil en monorepos para centralizar reglas comunes.

</details>

<details>
<summary>Diferencia entre transpilar y verificar tipos</summary>

Son dos tareas distintas. Verificar tipos exige entender todo el programa. Transpilar (quitar las anotaciones de tipo) se puede hacer archivo por archivo, y eso es lo que hacen esbuild o SWC dentro de Vite, sin revisar tipos. Por eso un proyecto puede ejecutarse aunque tenga errores de tipos: el bundler no los comprueba, y por eso conviene correr `tsc` (con `noEmit`) en el editor y en CI.

</details>

<details>
<summary>isolatedModules</summary>

Esta opción hace que TypeScript marque el código que no se podría transpilar de forma correcta archivo por archivo, algo que los transpiladores de un solo archivo (esbuild, SWC) no pueden resolver. Las plantillas de Vite y Next.js la activan. Su efecto práctico más visible es que, al reexportar un tipo, debes usar `export type`.

</details>

-----

## En entrevista

### Respuesta corta (junior)

`tsconfig.json` es el archivo de configuración de TypeScript. Indica qué archivos forman el proyecto (`include` y `exclude`) y qué reglas aplica el compilador (`compilerOptions`), por ejemplo `strict`, `target` y `jsx`. Lo usan `tsc`, el editor y otras herramientas, así todos aplican las mismas reglas.

### Respuesta ampliada (semi-senior)

* **Alcance:** `include`/`exclude`/`files` delimitan el proyecto. `exclude` solo filtra `include`; los archivos importados entran de todas formas.
* **Rigor:** `strict` agrupa varias opciones (`strictNullChecks`, `noImplicitAny`, entre otras). Se parte de `strict: true` y se relaja lo puntual.
* **Emisión:** `target` decide la sintaxis de salida y `lib` qué APIs se conocen. Son independientes: se puede emitir ES2022 y declarar solo `DOM`.
* **Módulos:** con bundler, `module: ESNext` y `moduleResolution: bundler`; en Node puro, `NodeNext`.
* **Papel de `tsc`:** con Vite o Next.js el bundler genera el JavaScript, así que `noEmit: true` deja a `tsc` como verificador de tipos.
* **Referencias:** la plantilla de Vite separa app (navegador) y herramientas (Node) en dos archivos, para que cada entorno tenga sus tipos. La raíz solo referencia.
* **Alias:** `paths` informa a TypeScript pero no al bundler, que necesita su propia configuración.

### Preguntas frecuentes de seguimiento

**1. ¿Qué diferencia hay entre `target` y `lib`?**
`target` es la versión de JavaScript que se emite. `lib` define qué APIs y tipos del entorno conoce el compilador. No agrega código al resultado.

**2. ¿Qué activa `strict`?**
Un conjunto de comprobaciones estrictas: `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, `strictPropertyInitialization`, `useUnknownInCatchVariables`, entre otras. Cada una se puede desactivar por separado.

**3. ¿Por qué Vite tiene tres archivos tsconfig?**
Porque el código de la app corre en el navegador y `vite.config.ts` corre en Node; cada uno necesita tipos distintos. La raíz solo referencia a los otros dos.

**4. ¿Para qué sirve `noEmit`?**
Para que `tsc` solo revise tipos y no genere `.js`. Se usa cuando otra herramienta (el bundler) se encarga de generar el JavaScript.

**5. ¿`paths` alcanza para tener un alias de importación?**
No. Solo lo entiende TypeScript. El bundler debe tener el mismo alias configurado; si no, el editor lo acepta pero la ejecución falla.

**6. ¿Qué hace `skipLibCheck`?**
Evita revisar los archivos `.d.ts`, incluidos los de `node_modules`. Acelera la compilación y evita conflictos de tipos entre librerías, sin afectar la revisión de tu código.

-----

## Siguiente lección

[Functions](03-Functions.md)
