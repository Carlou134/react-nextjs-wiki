# Calidad de código automatizada: Prettier, ESLint, commitlint, Lefthook y CI

## En una frase

La calidad de código no depende de acordarse de correr herramientas: se **automatiza en capas**. Lo rápido corre en cada commit, lo pesado en cada push y lo definitivo en CI, de modo que nada llega al repositorio sin formato, sin lint, sin tipos correctos y sin un mensaje de commit válido.

-----

## Antes de empezar

Conviene que ya conozcas:

* Cómo se crea y se estructura un proyecto con Vite: [Crear una app de React](01-Creating%20a%20React%20App.md).
* Qué es un commit, el *staging area* (`git add`) y un push.

Palabras usadas en esta nota:

* **Git hook:** script que git ejecuta automáticamente en un momento clave (antes de un commit, al escribir el mensaje, antes de un push). Si el script termina con un código distinto de `0`, git cancela la operación.
* **Archivos staged:** los archivos que agregaste con `git add` y que van a entrar en el próximo commit.
* **Exit code:** número con el que termina un programa. `0` significa éxito; cualquier otro valor, error. Los hooks y el CI no leen los mensajes: leen el exit code.
* **Conventional commits:** convención de mensajes con un tipo al inicio (`feat:`, `fix:`, `docs:`, `refactor:`, `chore:`...).
* **CI (integración continua):** servidor que vuelve a correr todas las verificaciones en cada pull request, por ejemplo con GitHub Actions.

-----

## El problema

Sin automatización, la calidad depende de la memoria:

1. **Formato inconsistente:** cada persona (o cada editor) formatea distinto, y los diffs se llenan de cambios de espacios y comillas.
2. **Errores que nadie ve:** código que no compila o que rompe una regla de lint puede quedar en el repositorio durante semanas, porque nadie corrió `tsc` ni `eslint`.
3. **Historial ilegible:** mensajes como `cambios varios` o `arreglé cosas` no dicen qué cambió ni por qué.

Caso real en react-lab: al agregar el script `typecheck` aparecieron **tres errores de TypeScript** y, más tarde, el primer `git push` con hooks encontró **tres errores de ESLint**. Todos existían desde antes; nadie los había visto porque nada los verificaba.

-----

## Cómo funciona: el embudo de capas

Cada capa es más lenta y más estricta que la anterior. La idea es detectar el error lo antes posible, donde es más barato de corregir.

| Capa | Cuándo corre | Sobre qué | Qué verifica |
| --- | --- | --- | --- |
| **pre-commit** | Al hacer `git commit` | Solo archivos staged | Formato (Prettier), lint con autocorrección (ESLint `--fix`), nombres de archivo |
| **commit-msg** | Justo después, al validar el mensaje | El mensaje del commit | Conventional commits (commitlint) |
| **pre-push** | Al hacer `git push` | Todo el proyecto | Lint completo y typecheck |
| **CI** | En cada pull request | Todo el proyecto, en una máquina limpia | Todo lo anterior + tests con cobertura + build |

Por qué el pre-commit trabaja solo con archivos staged: corre decenas de veces al día y tiene que ser **rápido**. Por qué el typecheck va en pre-push y no por archivo: un cambio en un archivo puede romper los tipos de otro que no tocaste, así que hay que revisar el proyecto entero.

Por qué el CI es indispensable aunque existan hooks: los hooks locales se saltan con `git commit --no-verify`. Son una red de seguridad para quien programa, no una garantía para el equipo. El CI no se puede saltar.

### Las herramientas

* **Prettier:** formatea el código. Con `prettier-plugin-tailwindcss` además ordena las clases de Tailwind.
* **ESLint:** detecta errores y malas prácticas (por ejemplo, dependencias faltantes en `useEffect`).
* **commitlint:** valida el mensaje del commit contra conventional commits.
* **Lefthook:** gestor de git hooks. Los hooks viven en `.git/hooks/`, una carpeta que **no se versiona**; Lefthook los declara en un archivo versionado (`lefthook.yml`) y los instala por ti.
* **Script de file-naming:** script propio que valida la convención de nombres de archivos y carpetas.

-----

## Mapa: lo que se usa en el trabajo vs lo que se hizo en react-lab

En el trabajo el proyecto es un monorepo con backend (Kotlin) y frontend. Esta tabla separa solo lo de **frontend** y lo compara con react-lab.

| Capa | En el trabajo | En react-lab | Estado |
| --- | --- | --- | --- |
| commit-msg | commitlint | commitlint + `@commitlint/config-conventional` | ✅ Hecho |
| pre-commit | Prettier sobre `*.{js,jsx,ts,tsx,css,json,mjs}` con `stage_fixed: true` | Prettier + plugin de Tailwind, igual | ✅ Hecho |
| pre-commit | ESLint `--fix` sobre `*.{js,jsx,ts,tsx,mjs}` | Igual | ✅ Hecho |
| pre-commit | `check-file-naming.mjs` | Script propio `scripts/check-file-naming.mjs` | ✅ Hecho |
| pre-push | `pnpm --filter frontend lint` | `pnpm lint` (sin `--filter`: no es monorepo) | ✅ Hecho |
| pre-push | `pnpm --filter frontend typecheck` | `pnpm typecheck` (`tsc -b`) | ✅ Hecho |
| CI | `frontend-quality`: lint, typecheck, `vitest run --coverage`, build | Pendiente | ⏳ Vitest instalado, falta configurarlo y el workflow |
| CI | `frontend-docker-build` | No aplica | ❌ El deploy sería en Vercel, que hace su propio build |

Lo que en el trabajo es **del backend o de todo el repositorio** y no se trasladó:

* `ktlintCheck`, `jacocoTestCoverageVerification` (gate de cobertura del 50 %), `backend-quality` y `backend-docker-build`: react-lab no tiene backend.
* `guard` (cierra automáticamente PRs de personas que no son owners hacia `develop`/`main`): no tiene sentido en un repositorio personal.
* `ci-passed`: job agregador que GitHub usa como *check* requerido. Útil cuando hay varios jobs; se puede sumar al armar el CI.

Detalle que explica una experiencia del trabajo: `stage_fixed: true` hace que, si Prettier o ESLint modifican un archivo durante el pre-commit, el resultado se vuelva a agregar al staging automáticamente. Por eso a veces el commit contiene cambios de formato que no se revisaron en el diff.

-----

## Qué se hizo en react-lab, paso a paso

### 1. Prettier

```bash
pnpm add -D prettier prettier-plugin-tailwindcss
```

`.prettierrc`:

```json
{
  "semi": false,
  "singleQuote": true,
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindStylesheet": "./src/index.css",
  "tailwindFunctions": ["cn", "clsx"]
}
```

* `semi` y `singleQuote` respetan el estilo que ya tenía el código, para que el primer formateo no cambie todos los archivos.
* `tailwindStylesheet` es obligatorio en Tailwind v4: sin él, el plugin no conoce los tokens propios del tema (`bg-surface`, `text-meta`) y no sabe ordenarlos.

`.prettierignore`: `dist`, `node_modules`, `pnpm-lock.yaml`.

### 2. Scripts en `package.json`

```json
"scripts": {
  "dev": "vite",
  "build": "tsc -b && vite build",
  "lint": "eslint .",
  "typecheck": "tsc -b",
  "format": "prettier --write .",
  "format:check": "prettier --check .",
  "check:naming": "node scripts/check-file-naming.mjs",
  "prepare": "lefthook install"
}
```

### 3. Convención de nombres y su script

La convención se **definió primero** a partir de lo que el código ya hacía, y recién después se automatizó:

| Qué | Convención | Ejemplo |
| --- | --- | --- |
| Carpetas de ejercicios | `NN-kebab-case` | `05-use-memo-callback` |
| Otras carpetas | kebab-case | `assets` |
| Componentes `.tsx` | PascalCase | `TaskCard.tsx` |
| HOCs y hooks en `.tsx` | camelCase con `with`/`use` | `withLoading.tsx` |
| Lógica `.ts` | camelCase | `cartStore.ts` |
| Tests | mismo nombre + `.test.ts(x)` | `dateRules.test.ts` |
| Entrada de cada ejercicio | `ExerciseNN.tsx`, con el número de la carpeta | `Exercise00.tsx` |
| Excepciones | se permiten tal cual | `main.tsx`, `App.tsx`, `index.css`, `README.md` |

El script (`scripts/check-file-naming.mjs`) tiene dos modos: sin argumentos recorre todo `src/`; con argumentos revisa solo esos archivos, que es lo que le pasa Lefthook. Normaliza las rutas de Windows (`\`) a `/` para funcionar igual en local y en CI (Linux), y termina con exit code `1` si encuentra errores.

Flujo aplicado: primero se corrió el script **en rojo** (detectó los cinco archivos de entrada con nombres distintos: `ExerciseComponent0`, `ComponentExercise3`...), después se renombraron con `git mv` para conservar el historial, y por último el script pasó **en verde**.

### 4. commitlint

```bash
pnpm add -D @commitlint/cli @commitlint/config-conventional
```

`commitlint.config.js` (con `export default` porque el proyecto usa `"type": "module"`):

```js
export default {
  extends: ['@commitlint/config-conventional'],
}
```

Se probó a mano antes de automatizarlo:

```bash
echo "arreglé cosas" | pnpm commitlint             # debe fallar
echo "fix: corrige el toggle" | pnpm commitlint    # debe pasar
```

### 5. Lefthook

```bash
pnpm add -D lefthook
pnpm lefthook install
```

`lefthook.yml`:

```yaml
pre-commit:
  parallel: true
  commands:
    prettier:
      glob: "*.{js,mjs,jsx,ts,tsx,css,json,md}"
      run: pnpm prettier --write {staged_files}
      stage_fixed: true
    eslint:
      glob: "*.{js,mjs,jsx,ts,tsx}"
      run: pnpm eslint --fix {staged_files}
      stage_fixed: true
    file-naming:
      glob: "src/*"
      run: node scripts/check-file-naming.mjs {staged_files}

commit-msg:
  commands:
    commitlint:
      run: pnpm commitlint --edit {1}

pre-push:
  parallel: true
  commands:
    lint:
      run: pnpm lint
    typecheck:
      run: pnpm typecheck
```

* `{staged_files}` se reemplaza por la lista de archivos staged que coinciden con el `glob`.
* `{1}` es el primer argumento que git le pasa al hook `commit-msg`: la ruta del archivo temporal con el mensaje.
* El script `prepare` vuelve a instalar los hooks después de cada `pnpm install`, por ejemplo al clonar el repositorio en otra máquina.

Se verificó con un commit inválido a propósito (`git commit -m "cambios varios"`), que tuvo que ser rechazado.

-----

## Errores comunes (los que aparecieron de verdad)

**Scripts fuera de `"scripts"`.** `format` y `format:check` quedaron en la raíz del `package.json`. pnpm no los ve y responde *command not found*. Van siempre dentro del objeto `"scripts"`.

**`tsc --noEmit` en un proyecto de Vite.** El `tsconfig.json` raíz de Vite no incluye archivos: solo tiene `references` a `tsconfig.app.json` y `tsconfig.node.json`. `tsc --noEmit` lee únicamente el raíz, no revisa nada y da un **falso verde**. Lo correcto es `tsc -b`, que sigue las referencias; no genera archivos porque ambos tsconfig ya tienen `"noEmit": true`.

**El prefijo `_` funciona en TypeScript pero no en ESLint.** TypeScript ignora parámetros como `_date` en `noUnusedParameters`, pero la regla `@typescript-eslint/no-unused-vars` no los ignora por defecto. Hay que alinearla:

```js
rules: {
  '@typescript-eslint/no-unused-vars': [
    'error',
    { argsIgnorePattern: '^_', varsIgnorePattern: '^_' },
  ],
},
```

Detalle: con la opción por defecto `args: 'after-used'`, un parámetro no usado **antes** de uno que sí se usa no se reporta. Por eso `_key` no fallaba y `_initialState` sí.

**`react-refresh/only-export-components`.** El HMR de Vite solo puede reemplazar en caliente un archivo que exporta **únicamente componentes**. Exportar además una constante (`SAMPLE_ITEMS`) o una función que no es componente (un HOC) obliga a recargar la página entera. Los `type` no cuentan porque desaparecen al compilar. Solución habitual: mover la constante a su propio archivo `.ts`. Si mezclarlos es intencional (por ejemplo, para comparar HOC y render prop lado a lado), se desactiva la regla en esa línea **con un comentario que explique por qué**.

**Nombres que solo difieren en mayúsculas, en Windows.** `withLoading.tsx` y `WithLoading.tsx` son el mismo archivo en Windows (y en macOS por defecto), porque el sistema de archivos no distingue mayúsculas. No pueden convivir en la misma carpeta.

**pnpm 10 y los scripts de instalación.** pnpm 10 bloquea por defecto los scripts `postinstall` de las dependencias, así que no conviene depender de que un paquete instale los hooks solo. El script `prepare` del propio proyecto sí se ejecuta.

**Stubs que no compilan.** Los archivos de arranque de ejercicios pendientes también pasan por el typecheck y el lint. `vitest` sin instalar, parámetros sin usar o imports sin usar bloquean el push aunque el ejercicio no se haya empezado.

-----

## Pendiente

1. **Vitest:** instalado. Falta la configuración y el script `test` (`vitest run`).
2. **GitHub Actions:** un workflow que en cada PR corra `lint`, `typecheck`, `check:naming`, `vitest run --coverage` y `build`, equivalente al job `frontend-quality` del trabajo.
3. **Deploy en Vercel:** opcional, para practicar.
4. **Wiki en Next.js:** replicar el mismo setup desde el día 1, sumando Vitest para la lógica que lee los `.md` y CI con `next build`.

-----

## Resumen en 5 líneas

1. La calidad se automatiza en capas: pre-commit (rápido, archivos staged), commit-msg, pre-push (proyecto entero) y CI (definitivo).
2. Lefthook versiona los git hooks en `lefthook.yml`; sin él viven en `.git/hooks/`, que no se comparte.
3. Primero se entiende y se prueba cada herramienta a mano; después se automatiza.
4. Una convención se define antes de validarla con un script, y el script se ve fallar antes de confiar en él.
5. Los hooks se saltan con `--no-verify`; por eso el CI es la única garantía real.

-----

## En entrevista

**Respuesta corta:** "Uso git hooks con Lefthook: en pre-commit Prettier y ESLint sobre los archivos staged, commitlint para conventional commits, y en pre-push lint y typecheck del proyecto completo. Después el CI repite todo y suma tests y build."

**Respuesta ampliada:** "Lo pienso como un embudo. Lo barato corre en cada commit y solo sobre lo staged, para no frenar el flujo. Lo que necesita ver todo el proyecto, como el typecheck, va en el pre-push. Y como cualquier hook se salta con `--no-verify`, el CI es la barrera real: corre en una máquina limpia, con cobertura mínima y build, y es un check requerido para mergear."

**Preguntas de seguimiento frecuentes:**

* *¿Por qué no correr el typecheck en pre-commit?* Porque es lento y necesita el proyecto entero: un cambio en un archivo puede romper los tipos de otro.
* *¿Qué problema tiene `stage_fixed`?* Lo que entra al commit puede no ser exactamente lo que revisaste en el diff, porque el hook lo modificó.
* *¿Lefthook o Husky + lint-staged?* Lefthook es un solo binario con un solo archivo de configuración y paraleliza los comandos; Husky necesita lint-staged para trabajar solo con archivos staged.

-----

## Siguiente lección

Con el código protegido por hooks, el paso siguiente es sumar tests con Vitest y un workflow de GitHub Actions que los corra en cada pull request.
