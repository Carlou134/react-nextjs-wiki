# ⚛️ React & Next.js Architecture Wiki

Repositorio personal de apuntes, guías rápidas (*cheatsheets*) y patrones de diseño para el desarrollo Frontend moderno con React, Next.js (App Router), TypeScript y gestión de estado.

---

## 📂 Estructura de Apuntes

### 1. React Core & TypeScript (`/01-react-core-and-typescript`)

- **00-typescript-fundamentals** — Types, tsconfig, funciones, arrays, tipos personalizados, union types, type narrowing y object types (TypeScript puro, sin React).
- **01-fundamentos-jsx** — JSX, Virtual DOM, JSX avanzado y multilínea.
- **02-componentes-y-props** — Primer componente, composición y Props.
- **03-tooling-y-devtools** — Vite/CRA y React Developer Tools.
- **04-hooks-y-context** — useState, useEffect, useLayoutEffect, Custom Hooks y Context API.
- **05-patrones-estilos-lifecycle** — HOC, Render Props, composición, estilos (CSS/Sass/styled-components/Tailwind) y ciclo de vida de clases.
- **06-forms** — Formularios controlados y componentes no controlados.
- **07-routing** — React Router v6.
- **08-manejo-de-errores** — Error Boundaries.
- **09-performance** — React Profiler, memoización y técnicas de optimización.
- **10-fetching-de-datos** — Fetch con useEffect, estados de carga/error, re-fetch por dependencias.
- **11-typescript-y-react** — Tipado de props, estado, useReducer y Context API.
- **12-react-19** — Server Components, Actions y los nuevos hooks de React 19.

### 2. Next.js App Router (`/02-nextjs-app-router`)
- Conceptos core, enrutamiento y Layouts.
- Server Components (RSC) vs. Client Components.
- Fetching de datos (paralelo, secuencial, Suspense, loading UI).
- Optimización.

### 3. State Management & Server State (`/03-state-management-and-data`)
- Zustand vs. Redux vs. Context: cuándo usar cada uno.
- Redux: core concepts, API, estado complejo, Redux Toolkit, middleware y thunks.
- React Query: caché, mutaciones y actualizaciones optimistas.

### 4. Testing & UI (`/04-testing-and-ui`) — *próximamente*
- Pruebas unitarias e integración de UI con **Vitest / Jest** y **React Testing Library**.
- Componentización accesible con **Tailwind CSS** y **Shadcn/UI**.

> La carpeta marcada *próximamente* todavía no tiene notas — se crea cuando el contenido correspondiente exista, para no dejar directorios vacíos en el repo.

---

## 🖼️ Imágenes

Las capturas e diagramas referenciados en las notas viven en `/Images` en la raíz del repo (rutas absolutas `/Images/archivo.ext`, así funcionan sin importar en qué subcarpeta esté la nota). Ver [Images/README.md](Images/README.md) para la lista de archivos pendientes de subir.
