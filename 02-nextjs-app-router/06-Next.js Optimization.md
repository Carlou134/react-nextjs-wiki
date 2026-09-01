# Next.js Optimization

## Optimizing with Next.js

Next.js ofrece características de optimización esenciales para mejorar el rendimiento del sitio web y las experiencias de usuario. Como desarrollador web, aprovechar estas características es clave para crear aplicaciones receptivas y eficientes.

Las siguientes son consideraciones a tener en cuenta al optimizar páginas web:

- **Mejorar Core Web Vitals**: Core Web Vitals, introducidos por Google, son tres métricas clave que miden la velocidad, interactividad y estabilidad visual de una página web. Las puntuaciones altas mejoran la experiencia del usuario y el SEO.
  - **Largest Contentful Paint (LCP)**: Mide el tiempo desde que un usuario navega a una página hasta que se muestra su elemento de contenido más grande. Las páginas ideales cargan su contenido más grande en menos de 2.5 segundos. El objetivo es reducir los tiempos de carga.
  - **Interaction to Next Paint (INP)**: Mide el tiempo que tarda la página en responder visualmente a una interacción del usuario, como hacer clic en un enlace o un botón. La página web ideal responde en menos de 200ms. El objetivo es mejorar la interactividad del usuario. INP reemplazó a **First Input Delay (FID)** como Core Web Vital de interactividad en marzo de 2024 — FID solo medía el retraso antes de que empezara a procesarse la primera interacción, mientras que INP mide la latencia completa de cualquier interacción durante toda la vida de la página, lo cual refleja mejor la experiencia real del usuario.
  - **Cumulative Layout Shift (CLS)**: Mide la mayor ráfaga de puntuaciones de cambio de diseño para cada cambio de diseño inesperado durante el ciclo de vida de una página. Un cambio de diseño ocurre cuando un elemento visible cambia de posición. Las páginas web ideales tendrán una puntuación CLS inferior a 0.1. El objetivo de CLS es mejorar la estabilidad visual.
- **Layout Shift**: Gestionar los medios de forma proactiva para asegurar que los elementos se carguen correctamente, permanezcan en su lugar y proporcionen una experiencia de usuario estable y libre de cambios.
- **Search Engine Optimization (SEO)**: Mejorar la clasificación en los motores de búsqueda de las páginas web con metadatos correctamente optimizados.
- **Escalabilidad y Reducción de Costos**: Optimizar recursos, reducir gastos generales innecesarios y mejorar diversas estrategias de desarrollo.
- **Optimización Móvil**: Asegurar que las experiencias de usuario sean consistentes en todos los dispositivos y tamaños de vista.
- **Mantenimiento del Desarrollador y Desarrollo Eficiente**: Código más limpio, mejor arquitectura y flujos de trabajo más fluidos.

Con esas consideraciones en mente para los siguientes ejercicios, utilizaremos las herramientas de optimización integradas de Next.js para optimizar:

- carga y redimensionado de imágenes
- carga y creación de fuentes
- importación y uso de scripts
- creación de metadatos

A medida que trabajes en esta lección, usaremos Lighthouse, una herramienta para evaluar Core Web Vitals y cuantificar las mejoras de optimización. Como el uso directo es limitado en los entornos de Codecademy, se te proporcionarán las puntuaciones esperadas de Lighthouse antes y después de la optimización para cada ejercicio.

-----

## Images and Priority

Optimizar páginas web puede reducir el LCP, mejorando significativamente la experiencia del usuario. Una lista amplia de elementos comunes que afectan esto incluye `<img>`, `<Image>`, `<video>`, `<url>`, llamadas `url()`, cualquier bloque de texto e imágenes animadas.

Recuerda, **LCP (Largest Contentful Paint)** es el tiempo desde que un usuario navega a una página hasta que se muestra su elemento de contenido más grande.

El LCP se calcula utilizando el tamaño visible de cada elemento. Las únicas excepciones son las imágenes. Si una imagen se redimensiona a un tamaño mayor, reportará su tamaño original. Si se redimensiona a un tamaño menor, reportará el tamaño más pequeño.

Un elemento solo se considera el LCP después de ser renderizado. El LCP puede cambiar a medida que se renderizan nuevos elementos. Si un elemento eliminado era el LCP, seguirá siendo el LCP a menos que se renderice un elemento más grande.

Similar al LCP es el **First Contentful Paint (FCP)**. El FCP mide el tiempo entre la carga de la página y la aparición del primer elemento. Las páginas web ideales cargarán su elemento FCP en menos de 1 segundo.

Aunque el FCP indica los tiempos de carga iniciales, los desarrolladores web se centran en el LCP debido a su impacto. El elemento FCP podría ser una etiqueta `<img>` vacía. Por el contrario, el LCP se centrará en el elemento de mayor impacto.

Next.js optimiza las imágenes utilizando su componente `<Image>`, que extiende el elemento HTML `<img>` con características para la optimización automática. Estas características de optimización incluyen:

- **Optimización de tamaño**: Las imágenes con el tamaño correcto reducen los cambios de diseño inesperados.
- **Cargas de página más rápidas**: Las imágenes solo se cargan cuando se muestran en la página web.
- **Flexibilidad de imágenes**: Puede redimensionar cualquier imagen según sea necesario.

En este ejercicio, nos centraremos en mejorar los tiempos de carga de imágenes y páginas. En el próximo ejercicio, nos centraremos en optimizar el dimensionado de imágenes.

Mostremos una imagen de manera adecuada y eficiente utilizando el componente `<Image>`. Así es como mostraríamos una imagen local:

```jsx
import Image from 'next/image';

function localImage() {
  return (
    <div>
      <h1>Imagen Local</h1>
      <Image
        src="Imagen Local.png"
        alt="Imagen Local"
      />
    </div>
  );
};

export default localImage;
```

Las imágenes remotas requieren atributos de ancho y alto especificados, y una propiedad `src` actualizada con la URL en línea.

```jsx
<Image
  src="Ubicación en Línea.png"
  alt="Imagen Local"
  width={500}
  height={500}
/>
```

Si sabemos que una imagen es el LCP, podemos asignarle la propiedad `priority`. Next.js priorizará esta `<Image>` y la precargará, reduciendo el tiempo total de carga.

```jsx
<Image
  ...
  priority={true}
/>
```

-----

## Image Sizes

Las imágenes se pueden optimizar aún más dimensionándolas adecuadamente para prevenir cambios de diseño (layout shifts) y mantener la relación de aspecto. El dimensionamiento de imágenes ocurre de tres formas: automática, explícita e implícita.

**Dimensionamiento automático** es dejar que la página web maneje el tamaño de la imagen sin establecer un ancho o alto. El dimensionamiento automático solo es aplicable a imágenes locales porque Next.js no puede deducir las dimensiones de imágenes remotas. Next.js utiliza importaciones estáticas para dimensionar automáticamente las imágenes locales.

```jsx
import Image from 'next/image'
import localImage from '../public/localImage.png'
 
export default function Page() {
  return (
    <Image
      src={localImage}
      alt="Imagen ubicada dentro de la carpeta del proyecto"
      {/* width={500} proporcionado automáticamente */}
      {/* height={500} proporcionado automáticamente */}
      {/* blurDataURL="data:..." proporcionado automáticamente */}
      {/* placeholder="blur" - Desenfoque opcional mientras se carga */}
    />
  )
}
```

**Dimensionamiento explícito** define tanto el ancho como el alto. El dimensionamiento explícito de Next.js es idéntico al dimensionamiento de `<img>` en HTML.

```jsx
import Image from 'next/image'
 
export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/mi-bucket/remoteImage.png"
      alt="Imagen almacenada remotamente en un bucket de AWS"
      width={500}
      height={500}
    />
  )
}
```

**Dimensionamiento implícito** estira las imágenes para llenar su contenedor padre. Next.js utiliza la propiedad `fill` para expandir la imagen y llenar su elemento padre.

```jsx
import Image from 'next/image'
 
export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/mi-bucket/remoteImage.png"
      alt="Imagen almacenada remotamente en un bucket de AWS"
      fill
    />
  )
}
```

Recuerda, para mejorar el LCP, debemos asegurarnos de que haya suficiente espacio para que una imagen no mueva inesperadamente a otros elementos.

La última consideración al optimizar imágenes es la ciberseguridad. Los actores maliciosos siempre están buscando interrumpir las páginas web. Afortunadamente, el componente `<Image>` contiene dos opciones de configuración principales para protegerse contra atacantes en línea: `remotePatterns` y `loaderFile`.

**remotePatterns**: Permite que las aplicaciones Next.js soliciten solo ciertos recursos o permitan ciertas rutas de directorio. Esta configuración permite comodines en la ruta si se solicitan imágenes con nombres variados.

Aquí hay un ejemplo que solo permite imágenes que utilizan el protocolo https accedidas desde `https://awsBucket123.com/misImagenes/`. Cualquier otro punto de acceso devolverá un error 400 Bad Request.

```js
/* next.config.js */
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'awsBucket123.com',
        port: '',
        pathname: '/misImagenes/**',
      },
    ],
  },
}
```

**loaderFile**: Permite que un desarrollador utilice un proveedor de nube para optimizar imágenes remotas en lugar de la API de optimización integrada de Next.js.

```js
/* next.config.js */
module.exports = {
  images: {
    loader: 'custom',
    loaderFile: './mi/image/loader.js',
  },
}
```

Dado el archivo de configuración anterior, tu loaderFile podría verse algo como lo siguiente:

```js
/* ./mi/image/loader.js */
'use client'

export default function myImageLoader({ src, width, quality }) {
  return `https://awsBucket123.com/${src}?w=${width}&q=${quality || 75}`
}
```

----

## Fonts

Las fuentes externas pueden afectar el rendimiento de las páginas web. Next.js optimiza las fuentes utilizando el módulo Font incorporado. El módulo Font descarga los archivos CSS y de fuentes necesarios en tiempo de construcción, preparando las fuentes con anticipación y evitando cambios de diseño innecesarios.

Next.js identifica y establece todas las fuentes del proyecto en tiempo de construcción; no se envían solicitudes de red adicionales.

Las fuentes no están disponibles globalmente para cada componente. Las fuentes son accesibles dependiendo de dónde se cree la fuente.

- Si se coloca en el layout raíz, está disponible en todas las rutas.
- Si se coloca en un layout diferente, está disponible en todas las rutas envueltas por ese layout.
- Si se coloca en una página individual, se precarga en la ruta única de esa página.

Las fuentes locales se crean utilizando la función `localFont()`. Las fuentes de Google se crean importándolas desde `'next/font/google'`. El siguiente es un ejemplo de cómo crear una fuente local y una fuente de Google utilizando el módulo Font.

Puedes consultar la lista de fuentes de Google en el repositorio de Next.js en GitHub para encontrar las fuentes de Google disponibles.

```jsx
/* Fuente Local */
import localFont from 'next/font/local'
 
const myFont = localFont({
  src: './some-font.woff2',
})

/* Fuente de Google */
import { Comfortaa } from 'next/font/google'
 
const comfortaa = Comfortaa({
  subsets: ['latin'],
})
```

Cada vez que se crea una fuente usando `localFont()` o `'next/font/google'`, se crea una nueva instancia de una fuente en una aplicación. Esto puede resultar en múltiples instancias de la misma fuente si se crea en varios archivos. En su lugar, podemos reutilizar una fuente, optimizando la aplicación Next.js. Hay 3 pasos para reutilizar fuentes.

1. Crear un cargador de fuentes en un archivo compartido
2. Exportar el cargador de fuentes como una constante
3. Importar la constante en cada archivo donde desees usar esta fuente

```jsx
/* Loader.ts */
import localFont from 'next/font/local'
 
export const spaceMono = localFont({
  src: '../public/fonts/SpaceMono-Bold.ttf',
  variable: '--font-SpaceMono-Bold',
})
```

```jsx
/* Page.tsx */
import { spaceMono } from 'Loader.ts';

export default function Home() {
  return <h1 className={spaceMono.className}>Página de Inicio</h1>;
}
```

----

## Scripts

Next.js también optimiza la importación de scripts de terceros con el componente `<Script>`. Este asegura que los scripts se carguen solo una vez, incluso si se hace referencia a ellos en múltiples archivos.

Recuerda, el **INP (Interaction to Next Paint)** es el tiempo que tarda una página web en responder visualmente a las interacciones del usuario.

El componente `<Script>` ayuda a mejorar el INP al reducir los tiempos de carga de los scripts. Los scripts son precargados por Next.js, preparándolos para ser ejecutados cuando sean llamados.

El siguiente es un ejemplo de cómo cargar un script.

```jsx
<Script
    src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"
></Script>
```

Podemos ajustar aún más cómo se cargan los scripts. Dependiendo de cuán ocupada esté una página web, los scripts se pueden cargar en diferentes momentos para asegurar que no bloqueen otros componentes, manteniendo la interactividad del usuario y optimizando el INP. Hay cuatro (4) formas de modificar la carga de scripts mediante la propiedad `strategy`.

- **beforeInteractive**: Carga el script antes de que se ejecute el código de Next.js y antes de la hidratación de la página.
- **afterInteractive**: (predeterminado) Carga el script inmediatamente después de que la página se vuelve interactiva, después de la hidratación inicial de la página.
- **lazyOnload**: Carga el script más tarde, durante el tiempo de inactividad del navegador.
- **worker**: Delega el manejo del script a un web worker (esto es experimental).

Puede ser útil saber cuándo se carga el script. En Next.js, hay tres (3) manejadores de eventos disponibles para el componente `<Script>`.

- **onLoad()**: Ejecuta código después de que el script se carga.
- **onReady()**: Ejecuta código después de que el script ha terminado de cargarse y cada vez que el componente se monta.
- **onError()**: Ejecuta código si el script falla al cargar.

```jsx
/* app/page.tsx */
'use client'
 
import Script from 'next/script'
 
export default function Page() {
  return (
    <>
      <Script
        src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"
        strategy="beforeInteractive"
        onLoad={() => {
          console.log('El script se ha cargado')
        }}
        onReady={() => {
          console.log('El script se ha cargado y el componente está montado')
        }}
        onError={() => {
          console.log('Error al cargar el script')
        }}
      />
    </>
  )
}
```

----

## Config-Based Metadata

La última característica de optimización es la **Metadata API**. La Metadata API ayuda a definir metadatos de aplicaciones web para mejorar el SEO (Search Engine Optimization) y generar tráfico web. Next.js optimiza los metadatos asistiendo en su generación.

Podemos definir un objeto Metadata de dos formas:

1. **Metadata basada en configuración (Config-based metadata)**
2. **Metadata basada en archivos (File-based metadata)**

Este ejercicio introducirá la metadata basada en configuración y el próximo ejercicio introducirá la metadata basada en archivos.

La metadata basada en configuración se centra en añadir metadatos a páginas individuales, como un título o descripción. Estos metadatos se crean utilizando un objeto Metadata estático o una función `generateMetadata()` dinámica en `layout.tsx` o `page.tsx`. Crearemos 2 instancias de metadata basada en configuración utilizando la Metadata API de Next.js.

**Objeto Metadata Estático**. La metadata estática se utiliza cuando el objeto de metadatos no cambiará.

```tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Título de la Página',
  description: 'Descripción de la Página',
} 

export default function Page() {}
```

**Objeto Metadata Dinámico**. Si un objeto Metadata es variable según la entrada, necesitaremos usar un objeto dinámico.

```tsx
import type { Metadata, ResolvingMetadata } from 'next'

export async function generateMetadata({ params }: { params: Promise<{ title: string }> }): Promise<Metadata> {
  const { title } = await params

  return {
    title,
  }
}
```

Al igual que en `page.tsx`, `params` es una `Promise` desde Next.js 15: `generateMetadata()` debe declararse `async` y resolverla con `await` antes de leer sus propiedades.

----

## File-Based Metadata

Next.js también utiliza **metadata basada en archivos** para optimizar la accesibilidad de las páginas web. La metadata basada en archivos optimiza los metadatos a través de archivos específicos:

- **robots.txt**: Información que muestra la estructura del sitio y permite el rastreo por parte de los motores de búsqueda.
- **sitemap.xml**: Ayuda a indexar sitios web e información sobre las líneas de tiempo de las páginas web.
- **favicon.ico, apple-icon.jpg, icon.jpg**: Optimizados para añadir iconos a las pestañas de las páginas web.
- **opengraph-image.jpg, twitter-image.jpg**: Ayudan con las imágenes cuando los usuarios comparten tu página web.

La metadata basada en archivos tiene mayor prioridad y anulará la metadata basada en configuración. Nos centraremos en `robots.txt` y `sitemap.xml` para crear 2 instancias de metadata basada en archivos utilizando la Metadata API de Next.js.

**robots.txt**: `robots.txt` es un archivo en el directorio raíz de la aplicación que indica a los rastreadores de los motores de búsqueda qué URLs están permitidas para acceder.

Podemos generar un `robots.txt` programáticamente con `robots.ts`. Aquí hay un ejemplo de un objeto Robots, indicando qué páginas están permitidas para ser rastreadas. Este ejemplo permite que los rastreadores accedan a cualquier ruta que comience con la ruta principal '/' mientras prohíbe el acceso a '/private'.

```ts
import { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: '/private/',
    },
    sitemap: 'https://paginaPrincipal.xml',
  }
}
```

**sitemap.xml**: `sitemap.xml` es un archivo especial que ayuda a los rastreadores de los motores de búsqueda a indexar un sitio de manera eficiente. Contiene información sobre la última actualización, la frecuencia de actualización y las relaciones entre todas las páginas del sitio.

Podemos generar un `sitemap.xml` programáticamente con `sitemap.ts`. Aquí hay un ejemplo de un objeto Sitemap que contiene todas las rutas de las páginas e información sobre las fechas de la última modificación, la frecuencia con que cambian y su nivel de prioridad. La propiedad `priority` indica cómo se comparan las páginas de la aplicación Next.js entre sí para los rastreadores de los motores de búsqueda. La prioridad predeterminada de las páginas es 0.5 y va de 0.0 a 1.0.

```ts
import { MetadataRoute } from 'next'

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    {
      url: 'https://paginaPrincipal.com',
      lastModified: new Date(),
      changeFrequency: 'yearly',
      priority: 1,
    },
    {
      url: 'https://paginaPrincipal.com/ruta1',
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.7,
    },
    {
      url: 'https://paginaPrincipal.com/ruta2',
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.4,
    },
  ]
}
```

La metadata se evalúa en un orden específico, comenzando desde el segmento raíz y avanzando hacia cada página. Los objetos Metadata exportados desde múltiples páginas en la misma ruta se combinan superficialmente para crear una salida de metadata final. Los objetos Metadata duplicados se eliminan según el orden.

----

## Review

¡Gran trabajo! Has aprendido a optimizar aplicaciones Next.js utilizando imágenes, scripts, fuentes y metadatos.

A lo largo de los ejercicios, se te presentó una puntuación de Lighthouse. En orden, esas puntuaciones fueron:

| Optimización | Puntuación Lighthouse |
|--------------|----------------------|
| Imágenes y Prioridad | 54 → 88 |
| Imágenes y Dimensionamiento | 88 → 95 |
| Fuentes | 95 → 97 |
| Scripts | 97 → 98 |
| Metadata Basada en Configuración | 98 → 98 |
| Metadata Basada en Archivos | 98 → 98 |

Como se señaló en el primer ejercicio, las imágenes impactan drásticamente en el LCP. Por el contrario, los metadatos tienen un impacto menor porque los archivos estaban preconstruidos con cambios menores a lo largo del ejercicio.

Hagamos un repaso:

- Las aplicaciones web pueden y deben optimizarse para minimizar el rendimiento de carga, los retrasos de interactividad y la estabilidad visual.
- **Largest Contentful Paint (LCP)** es el tiempo desde el inicio de la navegación hasta que el bloque de contenido más grande es visible para el usuario. Esto debería ocurrir dentro de 2.5 segundos cuando la página se carga inicialmente.
- **Interaction to Next Paint (INP)** es el tiempo desde que el usuario interactúa con la página, como un clic, hasta que el siguiente frame se pinta en pantalla. Esto no debería tardar más de 200ms. INP reemplazó a First Input Delay (FID) como Core Web Vital de interactividad en marzo de 2024.
- **Cumulative Layout Shift (CLS)** mide la mayor ráfaga de cambio de diseño por cada cambio inesperado durante la vida útil de una página. Esto debería puntuar menos de 0.1.
- Next.js tiene herramientas integradas para optimizar imágenes, fuentes, scripts y metadatos.
- Next.js contiene un componente `<Image>` incorporado que extiende el elemento HTML `<img>` con características para la optimización automática.
- Next.js contiene un módulo Font incorporado para agregar fuentes web sin cambios de diseño y sin solicitudes a Google.
- Next.js contiene un componente `<Script>` incorporado para ayudar a cargar scripts de terceros, cargándolos solo una vez, incluso si el usuario navega entre páginas.
- Next.js tiene una **Metadata API** para ayudar a definir metadatos de la aplicación para mejorar el SEO y la capacidad de compartir.
- La **metadata basada en configuración** exporta un objeto metadata a un archivo `layout.tsx` o `page.tsx`.
- La **metadata basada en archivos** agrega un archivo generado estática o dinámicamente a una ruta.
- **Lighthouse** es una herramienta automatizada de código abierto para medir y evaluar el rendimiento web. Su objetivo principal es mejorar la experiencia general de un sitio web.

Ideas:

- Optimiza aún más las aplicaciones usando el plugin [bundle-analyzer](https://nextjs.org/docs/app/guides/package-bundling) para Next.js. Este plugin te ayuda a manejar el tamaño de los módulos de Javascript.
- Haz seguimiento de la analítica de la página web usando [Next.js analytics](https://nextjs.org/docs/app/guides/analytics) para gestionar tus propias métricas de rendimiento.
- Administra los metadatos en proyectos grandes usando varios archivos sitemap.ts.

-----
