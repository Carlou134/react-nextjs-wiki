# Next.js Core Concepts

## Rendering Environments

En los últimos años, Next.js ha ganado una popularidad significativa en la comunidad de desarrollo web por sus técnicas de renderizado versátiles y flexibles. Para comprender las capacidades de Next.js y entender por qué es una opción preferida para las aplicaciones web modernas, debemos comenzar desde el núcleo: los entornos de renderizado.

El renderizado es el proceso de convertir código en una visualización visual e interactiva que los usuarios pueden ver e interactuar dentro de un navegador web. Este proceso comienza cuando un navegador solicita una página web y termina con la respuesta del servidor, culminando en la aplicación renderizada con la que el usuario interactúa.

El proceso de renderizado es una consideración crucial para cualquier desarrollador web. La ubicación del proceso puede afectar varios aspectos, como la experiencia general del usuario, el rendimiento de la aplicación web y la visibilidad en los motores de búsqueda para generar tráfico orgánico al sitio.

Existen dos entornos de renderizado principales: servidor y cliente. El renderizado del lado del servidor (SSR) significa que el ensamblaje de la página web ocurre principalmente en el servidor, mientras que el renderizado del lado del cliente (CSR) se ensambla principalmente en el navegador del cliente. Una aplicación web bien optimizada utiliza una combinación de ambos métodos, aprovechando las fortalezas de cada uno.

Si bien React es compatible con ambos, carece de SSR incorporado. Esto hace que Next.js sea una opción ideal para los desarrolladores, ya que ofrece un soporte sólido tanto para SSR como para CSR. Con Next.js, podemos especificar la granularidad del renderizado hasta el nivel de componente, eligiendo si debe ser renderizado en el servidor, en el cliente o una combinación de ambos.

En los siguientes dos ejercicios, exploraremos en detalle los conceptos de renderizado del lado del servidor y renderizado del lado del cliente.

### 🧠 Diferencias clave (bien claras)

#### 🔹 Client-Side Rendering (CSR)

👉 (React normal, Vite)

* HTML inicial vacío
* El navegador construye todo
* ⏳ Carga inicial lenta
* ⚡ Luego va rápido (SPA)
* ❌ SEO no tan bueno

#### 🔹 Server-Side Rendering (SSR)

👉 (Next.js)

* HTML ya viene listo del servidor
* 🚀 Carga inicial rápida
* ✅ Mejor SEO
* 😅 Más carga en el servidor

### ⚔️ Diferencia en una frase (para entrevistas)

👉 **CSR renderiza en el navegador, SSR en el servidor. SSR mejora la carga inicial y SEO, mientras CSR mejora la interactividad después de cargar.**

👉 SSR se usa para la primera carga, CSR para la interacción posterior.

---

## Client-Side Rendering

Piensa en un sitio web que ofrece una experiencia de usuario altamente interactiva, fluida y dinámica como YouTube y AirBnB. El renderizado del lado del cliente (CSR) juega un papel importante en hacer posibles este tipo de experiencias.

En CSR, la respuesta del servidor al cliente incluye todos los archivos necesarios para renderizar los componentes en el navegador del cliente y habilitar la interactividad sin necesidad de realizar solicitudes adicionales al servidor para el renderizado ni recargar la página. La respuesta contiene una página HTML básica y el JavaScript que la acompaña ensambla el resto del contenido de la página.

CSR es una solución ideal para componentes con estado o muchas interacciones de usuario como botones y campos de formulario. Una implementación popular de CSR es el patrón de aplicación de una sola página (SPA). En este patrón, el usuario permanece en una sola página, mientras que JavaScript actualiza o reemplaza el contenido de manera fluida. Este enfoque no recarga la página por completo, sino que obtiene dinámicamente nuevos datos externos del servidor según sea necesario — piensa en cómo tu bandeja de entrada de Gmail se actualiza con nuevos correos electrónicos, todo sin recargar la página en sí.

En Next.js, el renderizado del lado del cliente se puede implementar explícitamente a través de componentes cliente, una característica opcional que permite a los desarrolladores designar componentes específicos para ser renderizados en el cliente.

En una lección posterior, profundizaremos en los detalles de los componentes cliente, pero por ahora, debes saber que puedes definir un componente cliente como lo harías con un componente React normal con una directiva 'use client'. Esta directiva especifica que el componente y sus componentes hijos deben ser renderizados en el lado del cliente.

El siguiente es un ejemplo de un componente renderizado del lado del cliente:

```jsx
'use client'
import React, { useState } from 'react'

export default function Page() {
  const [toggle, setToggle] = useState<boolean>(false);
  return (
    <div onClick={() => setToggle(!toggle)}>
      {toggle ? 'Verdadero': 'Falso'}
    </div>
  )
}
```

-----

## Server-Side Rendering

Si bien el renderizado del lado del cliente permite que los sitios web sean dinámicos e interactivos, construir y cargar la página completamente en el hardware del cliente puede ser una tarea costosa. Una conexión a internet lenta o un hardware de cliente lento pueden aumentar el tiempo de espera para el usuario, que se queda mirando una pantalla vacía mientras el navegador ensambla la página web.

En el renderizado del lado del servidor (SSR), la página web se ensambla en el servidor. Transferir la carga del renderizado al servidor aprovecha una infraestructura de servidor más potente, reduciendo la carga en el hardware del cliente.

El renderizado del lado del servidor es ideal para aplicaciones web que necesitan mucha obtención de datos, optimización para motores de búsqueda (SEO) y velocidad. Al acercar las solicitudes de obtención de datos a la base de datos, los desarrolladores redujeron la latencia de estas solicitudes. Enviar páginas completamente renderizadas también significa que los usuarios pueden verlas inmediatamente al visitar un sitio web, independientemente de las capacidades de su hardware. Las páginas renderizadas luego pueden ser rastreadas e indexadas por los bots de los motores de búsqueda, lo que lleva a un mejor SEO.

Por defecto, Next.js renderiza los componentes en el lado del servidor. Puedes definir un componente React sin configuraciones adicionales, ya que Next.js maneja automáticamente el renderizado del lado del servidor.

Dentro del propio renderizado del lado del servidor, Next.js ofrece tres enfoques distintos: renderizado estático, renderizado dinámico y streaming. En una lección posterior sobre Componentes de Servidor, profundizaremos en estos subconjuntos y sus aplicaciones específicas.

En esta etapa, aunque la página es visible y contiene elementos como botones y campos de formulario, no son completamente interactivos. En cierto sentido, es como una primera capa de pintura fresca en nuestra página. En el próximo ejercicio, hablaremos sobre lo que viene después para hacer que la aplicación web sea interactiva.

-----

## Adding Interactivity With Hydration

Anteriormente, hablamos sobre el renderizado del lado del servidor y señalamos que, si bien nuestra aplicación web es visible, no es interactiva. Profundicemos en la hidratación y refinemos nuestro modelo mental de cómo opera Next.js.

Si el proceso de SSR es la primera capa de pintura, la segunda capa es la hidratación. Después de que el HTML enviado por el servidor se carga en el navegador del cliente, el paquete de JavaScript que acompaña al HTML comienza a ejecutarse. Este JavaScript incluye el código de React, que luego "hidrata" el HTML estático. La hidratación implica adjuntar controladores de eventos y vincular los componentes de React con sus contrapartes HTML. Durante este proceso, React también realiza la reconciliación, comparando el resultado de renderizar los componentes en el lado del cliente con el resultado de renderizar en el servidor, asegurando que estén sincronizados.

Una vez que se completa la hidratación, la página web se vuelve completamente interactiva. Los elementos interactivos como botones y formularios ahora pueden responder a las entradas del usuario. A partir de este punto, cualquier actualización en la página, como interacciones del usuario u obtención de datos, conduce a un nuevo renderizado de los componentes afectados. Este nuevo renderizado se maneja completamente en el lado del cliente, y los componentes se actualizan para reflejar nuevos estados o props. ¡Considérelo como un retoque de las capas de pintura!

Con la hidratación, podemos ver que el renderizado con Next.js no se limita a un solo lado. Una aplicación Next.js típicamente representa una combinación de diferentes técnicas de renderizado, aprovechando las fortalezas del servidor y del cliente para ofrecer una experiencia de usuario óptima.

----

## Setting Up a Project

Con los conceptos de CSR, SSR e hidratación ya vistos, podemos comenzar a configurar un proyecto.

La forma más rápida de empezar a trabajar en tu primer proyecto Next.js es usar la herramienta CLI create-next-app. Los creadores de Next.js la mantienen oficialmente y admite la inicialización de aplicaciones Next.js con varias plantillas. Un beneficio significativo de esta característica de inicialización es la capacidad de crear una aplicación Next.js con configuraciones específicas a partir de solo una URL de repositorio de GitHub.

create-next-app tiene una experiencia de configuración interactiva donde las indicaciones te permiten personalizar tu proyecto. No hay un conjunto único de respuestas para las siguientes preguntas; las elecciones que hagas dependerán de tus necesidades y preferencias específicas para el proyecto.

```
¿Cómo se llama tu proyecto?  <nombre_del_proyecto>
¿Te gustaría usar TypeScript?  No / Sí
¿Te gustaría usar ESLint?  No / Sí
¿Te gustaría usar Tailwind CSS?  No / Sí
¿Te gustaría usar el directorio `src/`?  No / Sí
¿Te gustaría usar App Router? (recomendado)  No / Sí
¿Te gustaría personalizar el alias de importación predeterminado (@/*)?  No / Sí
```

create-next-app también acepta argumentos de línea de comandos. Normalmente, puedes especificar un nombre de proyecto usando este comando: `create-next-app ecommerce-site`. Esta acción establecerá un proyecto Next.js titulado ecommerce-site.

Para inicializar la aplicación con un ejemplo existente de la colección de ejemplos de Next.js o una URL de GitHub, añade la bandera `--example` al comando.

Cuando se usa create-next-app, genera una estructura básica de proyecto y configuración. Un componente clave de esta estructura es la carpeta `app`. Aquí es donde pasarás la mayor parte de tu tiempo. Junto con la carpeta `app`, el proyecto tendrá un conjunto de archivos fuente y de configuración.

El comando `npm run dev`, que hemos usado para iniciar nuestro servidor de desarrollo en los dos ejercicios anteriores, está configurado para ejecutar el script dev preconfigurado por create-next-app.

-----

## The App Router

Para este curso, nuestra configuración de proyecto utilizará el App Router. El App Router es un nuevo Router disponible listo para usar en el directorio `/app` de tu proyecto Next.js. Esta característica de Next.js aborda un problema significativo en React: la falta de una implementación nativa de enrutamiento. Simplifica el desarrollo web al proporcionar un sistema de enrutamiento potente y flexible, obtención eficiente de datos, rendimiento mejorado y fácil adopción en aplicaciones existentes.

El App Router es un enrutador basado en sistema de archivos donde la estructura de tu directorio `/app` determina las rutas y las rutas URL disponibles para toda tu aplicación. Exploraremos el enrutamiento más profundamente en la próxima lección. Por ahora, exploraremos el directorio `/app` y cómo genera rutas básicas.

En el App Router, cada nombre de carpeta determina una ruta que existe. Para hacer accesible la ruta, debe existir un archivo `page.tsx` en el directorio.

Por ejemplo, la carpeta raíz `/app` puede tratarse como una página de inicio y agregar un `page.tsx` la hace accesible; la página se corresponderá con la ruta URL `/`. La interfaz de usuario de la página de inicio, o de cualquier página en una ruta, está contenida en el archivo `page.tsx`.

El contenido de `page.tsx` se vería así:

```tsx
export default function Page() {
  return <h1>Hello, World!</h1>
}
```

Este es un componente Page exportado por defecto que contendrá la interfaz de usuario para la ruta `/`, que contendrá un encabezado que dice "Hello, World!"

-----

## Styling in Next.js

Una aplicación Next.js se puede estilizar de varias formas diferentes. Por defecto, Next.js tiene soporte incorporado para CSS Global, CSS Modules, Tailwind CSS, CSS-in-JS y Sass.

En este ejercicio, usaremos CSS Modules y CSS Global.

CSS Modules es una excelente manera de prevenir colisiones de nombres de estilos mediante la modularización de archivos CSS. Los archivos se procesarán como un CSS Module si se les añade la extensión `.module.css`. En cuanto a la ubicación, los archivos CSS Module pueden estar en el mismo lugar que el componente que están estilizando.

Por ejemplo:

```jsx
import styles from './HomePage.module.css'

export default function Page() {
  return <h1 className={styles.header}>Hello, World!</h1>
}
```

Los archivos CSS Global pueden aplicar un estilo a toda la aplicación. El archivo CSS global se puede importar en cualquier página o layout. Un layout es un componente especial que se detallará en la próxima lección de Routing, pero típicamente, el archivo CSS global se importa en el layout raíz, un archivo layout que se encuentra en el directorio `/app`.

Por ejemplo:

```tsx
// layout.tsx
import './global.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

----

## Review

¡Bien hecho! Has llegado al final de esta lección sobre los conceptos fundamentales de Next.js.

Antes de terminar, aquí tienes un resumen:

- **Next.js** es un framework de React que permite crear aplicaciones web de alto rendimiento, escalables y amigables con los motores de búsqueda.
- El **renderizado** es el proceso de convertir código en interfaces de usuario visibles.
- Next.js es compatible con el **renderizado del lado del cliente** y el **renderizado del lado del servidor**. Permite a los desarrolladores crear aplicaciones híbridas donde partes de la aplicación pueden estar en el servidor y partes en el cliente.
- El **renderizado del lado del cliente** envía las instrucciones al cliente para que sean renderizadas en el navegador.
- El **renderizado del lado del servidor** renderiza las páginas web en el servidor y las envía al navegador del cliente.
- La **hidratación** hace que las páginas sean interactivas al adjuntar JavaScript a los componentes correspondientes.
- El **App Router** utiliza el sistema de archivos para proporcionar enrutamiento a una aplicación Next.js.
- **create-next-app** se puede utilizar para iniciar rápidamente una aplicación Next.js.
- Next.js es compatible con el estilizado mediante **CSS global**, **CSS Modules**, **CSS-in-JS**, **Tailwind CSS** y **Sass**.

-----

## Comando para crear proyecto en next.js

```bash
npx create-next-app my-app
```

----