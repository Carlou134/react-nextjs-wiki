# Atomic Design

## Un problema distinto: cómo organizar los componentes de UI en sí

La lección anterior habla de cómo organizar un proyecto por **funcionalidad de negocio** (Vertical Slice) versus por **tipo técnico de archivo** (horizontal). **Atomic Design** resuelve un problema relacionado pero distinto: no cómo organizar el proyecto completo, sino específicamente **cómo estructurar los componentes de interfaz visual** dentro de una librería de componentes compartidos (por ejemplo, dentro de la carpeta `shared/components/` que vimos antes), en función de su nivel de complejidad.

Atomic Design es una metodología propuesta por el diseñador Brad Frost, tomada prestada del vocabulario de la química: así como los átomos se combinan para formar moléculas, y las moléculas se combinan para formar organismos, los componentes de UI más simples se combinan para formar componentes cada vez más complejos. Define cinco niveles.

-----

## Los cinco niveles

### Átomos

Los **átomos** son los bloques de construcción más básicos de una interfaz: no se pueden descomponer en partes más pequeñas sin dejar de tener sentido por sí solos. Un botón, un input de texto, una etiqueta, un ícono, un label.

```tsx
// átomo: Button.tsx
function Button({ children, ...props }: ButtonProps) {
  return <button className="btn" {...props}>{children}</button>;
}
```

Un átomo, en general, no sabe nada sobre el contexto de negocio en el que se va a usar — un `Button` no sabe si va a servir para "Agregar al carrito" o para "Cerrar sesión". Por eso, los átomos son los candidatos más naturales para vivir en `shared/components/`.

### Moléculas

Las **moléculas** son grupos simples de átomos que funcionan juntos como una unidad. Un campo de búsqueda (un `Input` de texto más un `Button` de buscar) es una molécula típica: cada pieza sigue siendo reconocible por separado, pero juntas cumplen una función que ninguna de las dos cumple sola.

```tsx
// molécula: SearchBar.tsx
function SearchBar({ onSearch }: SearchBarProps) {
  const [query, setQuery] = useState('');

  return (
    <div className="search-bar">
      <Input value={query} onChange={(e) => setQuery(e.target.value)} />
      <Button onClick={() => onSearch(query)}>Buscar</Button>
    </div>
  );
}
```

### Organismos

Los **organismos** son secciones más complejas de la interfaz, compuestas por varias moléculas y/o átomos combinados. Un encabezado de sitio (logo, navegación, `SearchBar`, ícono de usuario) es un organismo típico: es una pieza con identidad propia y reconocible, pero sigue estando construida enteramente a partir de piezas más pequeñas ya definidas.

```tsx
// organismo: Header.tsx
function Header() {
  return (
    <header>
      <Logo />
      <NavMenu />
      <SearchBar onSearch={handleSearch} />
    </header>
  );
}
```

A esta altura, los organismos suelen empezar a tener alguna noción de contexto de negocio (un `Header` de e-commerce no es igual al de una app de banca), lo que los hace menos genéricos que los átomos y moléculas — por eso es común que empiecen a vivir más cerca de una `feature/` específica que de `shared/`, salvo que ese organismo en particular se reutilice literalmente en toda la aplicación.

### Templates

Los **templates** definen la **estructura de una página** —dónde va el header, dónde va el contenido principal, dónde va el sidebar— combinando organismos, sin llenar todavía esos espacios con datos reales. Un template describe el layout, no el contenido.

```tsx
// template: DashboardTemplate.tsx
function DashboardTemplate({ sidebar, mainContent }: DashboardTemplateProps) {
  return (
    <div className="dashboard-layout">
      <Header />
      <aside>{sidebar}</aside>
      <main>{mainContent}</main>
    </div>
  );
}
```

### Pages (páginas)

Las **páginas** son instancias concretas de un template, con datos reales conectados — la versión final que efectivamente ve el usuario. Una página toma un template y lo llena con contenido específico, generalmente obtenido de una API o de un store de estado.

```tsx
// página: DashboardPage.tsx
function DashboardPage() {
  const { data: stats } = useQuery({ queryKey: ['stats'], queryFn: fetchStats });

  return (
    <DashboardTemplate
      sidebar={<SidebarMenu />}
      mainContent={<StatsPanel stats={stats} />}
    />
  );
}
```

-----

## Cómo se combina con Vertical Slice

Atomic Design y Vertical Slice no compiten entre sí — resuelven preguntas distintas y se pueden usar juntos sin conflicto:

* **Vertical Slice** responde: *¿en qué carpeta del proyecto vive este código, según a qué funcionalidad de negocio pertenece?*
* **Atomic Design** responde: *¿qué tan complejo es este componente de UI en particular, y de qué piezas más simples está hecho?*

Una combinación común es aplicar Atomic Design **dentro** de `shared/components/`, para los átomos y moléculas genuinamente reutilizables en toda la aplicación, mientras que los organismos, templates y páginas específicos de una funcionalidad de negocio (que ya conocen el dominio: "carrito de compras", "perfil de usuario") viven dentro de la `feature/` correspondiente:

```
src/
├── shared/
│   └── components/
│       ├── atoms/
│       │   ├── Button.tsx
│       │   └── Input.tsx
│       └── molecules/
│           └── SearchBar.tsx
└── features/
    └── cart/
        ├── components/          # organismos específicos del carrito
        │   └── CartSummary.tsx
        └── pages/                # páginas del carrito
            └── CheckoutPage.tsx
```

-----

## Cuándo vale la pena (y cuándo no)

Atomic Design aporta un vocabulario compartido muy útil para hablar sobre la complejidad relativa de los componentes (todo el equipo entiende qué significa "esto es una molécula, no un átomo"), y ayuda a detectar cuándo un componente se volvió demasiado grande para su propio nivel: un "átomo" que termina con quince props y lógica condicional compleja probablemente debería ser, en realidad, una molécula.

Dicho esto, adoptar los cinco niveles de forma rígida —creando literalmente carpetas `atoms/`, `molecules/`, `organisms/`, `templates/` y `pages/` desde el primer día de un proyecto chico— puede generar la misma sobre-ingeniería que cualquier otra abstracción aplicada antes de que el proyecto la necesite. El valor real de Atomic Design no está en respetar la taxonomía al pie de la letra, sino en el hábito mental que promueve: pensar la interfaz como una jerarquía de piezas reutilizables que se combinan, en lugar de escribir cada página como un bloque monolítico de JSX.
