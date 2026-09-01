# Next.js Server Components

## Introduction to Server Components

Los desarrolladores deben considerar constantemente el renderizado y la gestión de componentes.

Recordemos que el Renderizado del Lado del Servidor (SSR), a cambio de interactividad, tiene cargas iniciales de página más rápidas y beneficios para el SEO. Por el contrario, una aplicación web renderizada del lado del cliente permite una interactividad más rica a costa de cargas iniciales de página más lentas y posibles desventajas para el SEO.

Entonces llegaron los Componentes de Servidor, también conocidos como React Server Components (RSCs). Hemos hablado anteriormente sobre los Componentes de Servidor, señalando que son el valor predeterminado para todos los componentes en Next.js, que ha integrado toda la arquitectura de React para Componentes de Servidor. Estos Componentes de Servidor se renderizan en el servidor, optimizando los tiempos de carga y la eficiencia del SEO. Diferenciándose del renderizado tradicional del lado del servidor, los Componentes de Servidor permiten una integración más dinámica de elementos del lado del cliente. Esto permite una mayor flexibilidad, mejorando la interactividad y el compromiso del usuario sin sacrificar el rendimiento.

Antes de continuar, recalibremos y resolvamos un malentendido común: los Componentes de Servidor no reemplazan el Renderizado del Lado del Servidor. Se complementan entre sí y trabajan juntos en la caja de herramientas del desarrollador. Piénsalo así:

- Los Componentes de Servidor operan a nivel de componente
- El Renderizado del Lado del Servidor opera a nivel de página

Si bien ambos renderizan contenido en el servidor, los Componentes de Servidor se centran en descargar cierta lógica a nivel de componente al servidor, mientras que el SSR se ocupa de renderizar la página completa en el servidor.

Supongamos que estamos creando una aplicación web que muestra artículos. Usando SSR, la página completa de un artículo se generaría en el servidor y se enviaría al cliente en un gran fragmento, donde el lado del cliente toma el control y hidrata la página. En contraste, usando Componentes de Servidor, cada sección del renderizado del artículo puede descargarse al servidor o al cliente según sea necesario. Por ejemplo, un componente de servidor podría manejar el renderizado inicial de una parte estática de la página, y un componente cliente podría tomar las partes dinámicas de la página que tienen interacción del usuario.

En consecuencia, los Componentes de Servidor representan un cambio de paradigma que nos permite controlar granularmente el entorno de renderizado de cada componente, construyendo aplicaciones web híbridas y de alto rendimiento.

En esta lección, nos centraremos en los Componentes de Servidor de Next.js; cubriremos:

- cómo se renderizan los Componentes de Servidor
- estrategias de renderizado en servidor con Componentes de Servidor
- cómo implementar Componentes de Servidor
- uso de Componentes Fallback con React Suspense
- uso de Componentes Cliente con Componentes de Servidor
- explorar cómo se diferencian los Componentes de Servidor y los Componentes Cliente

----

## How Server Components are Rendered

Si bien hemos afirmado que los Componentes de Servidor representan un cambio de paradigma pragmático — donde los desarrolladores pueden establecer una división del trabajo más precisa entre el servidor y el cliente — aún no hemos hablado mucho sobre cómo se renderizan los Componentes de Servidor en el contexto de una aplicación Next.js.

Este proceso es una coordinación sofisticada entre el servidor y el cliente. Cuando una solicitud llega al servidor Next.js para una ruta determinada:

1. **Next.js prepara el entorno** para renderizar los componentes React asociados con la ruta solicitada.

![paso-1](/Images/2197-01-SCB1Hqb1.svg)

Next.js prepara el entorno para renderizar los componentes de React asociados a la ruta solicitada.

2. **Next.js divide el trabajo de renderizado** en unidades más pequeñas llamadas fragmentos (chunks). Los fragmentos están determinados por los segmentos de ruta — cada parte de la ruta que puede considerarse separada a efectos de renderizado — y cualquier límite de Suspense.

![paso-2](/Images/2197-02-b4uQMo78.svg)

Next.js divide el trabajo de renderizado en unidades más pequeñas llamadas fragmentos.

3. **Cada fragmento se procesa en dos pasos:**
   - React renderiza los Componentes de Servidor en un formato conocido como **React Server Component Payload (RSC Payload)**. Este payload es una representación binaria compacta de la salida renderizada de los Componentes de Servidor, marcadores de posición para Componentes Cliente, y cualquier prop necesaria de los Componentes de Servidor a los Componentes Cliente.
   - El RSC Payload trabaja con JavaScript para renderizar HTML en el servidor.

![paso-3](/Images/2197-03-Cd7AZGkx.svg)

Cada fragmento se procesa en dos pasos: React renderiza los componentes del servidor en un formato conocido como React Server Component Payload (RSC Payload). A continuación, el RSC Payload trabaja con JavaScript para renderizar HTML en el servidor.

4. **El servidor envía el HTML y el RSC payload** al cliente.

![paso-4](/Images/2197-04-ebCmN6Ao.svg)

El servidor envía el HTML y la carga útil RSC al cliente.

5. **React utiliza el RSC Payload** en el cliente para actualizar el DOM del navegador, alineando los componentes renderizados en el servidor con sus contrapartes del lado del cliente. Esto actualiza el estado de los componentes y carga los Componentes Cliente donde estaban los marcadores de posición.

![paso-5](/Images/2197-05--S1Hg8z_.svg)

React utiliza la carga útil RSC en el cliente para actualizar el DOM del navegador, alineando los componentes renderizados en el servidor con sus contrapartes del lado del cliente.

6. **Se utiliza JavaScript para hidratar** los Componentes Cliente y dotarlos de interactividad.

![paso-6](/Images/2197-06-_XyjSJnk.svg)

JavaScript se utiliza para dar vida a los componentes del cliente y permitir la interactividad.

Al comprender el proceso de renderizado de los Componentes de Servidor, los desarrolladores pueden tomar decisiones más informadas sobre las estrategias de renderizado y el ciclo de vida de un componente.

----

## Server Rendering Strategies

Siguiendo la filosofía de Next.js de optimizar el desarrollo web, ofrece tres estrategias de renderizado en servidor para adaptarse a diferentes necesidades: estático, dinámico y streaming, cada una con enfoques personalizados de caché y entrega de contenido.

El **renderizado estático** pre-genera páginas en tiempo de construcción, permitiendo que los resultados se almacenen en caché y se distribuyan eficientemente a través de una Red de Entrega de Contenidos (CDN). Este método es ideal para contenido que permanece sin cambios a lo largo del tiempo, reduciendo significativamente la carga del servidor y mejorando la velocidad de entrega al servir contenido estático y pre-renderizado a todos los usuarios.

El **renderizado dinámico**, por otro lado, genera contenido en tiempo real para cada solicitud, atendiendo a necesidades de contenido personalizado o sensible al tiempo. Este enfoque garantiza la frescura del contenido pero requiere más recursos en comparación con el renderizado estático, ya que omite la caché para generar datos dinámicamente para cada usuario, potencialmente ralentizando el tiempo de respuesta.

Tanto los métodos estáticos como dinámicos se ven influenciados por el rendimiento de las operaciones de obtención de datos, donde las demoras en la obtención pueden afectar la velocidad general del renderizado de la página.

El **streaming** divide la interfaz de usuario de una ruta en fragmentos que se transmiten progresivamente al cliente. Este método mejora significativamente los tiempos de carga percibidos, ya que los usuarios pueden interactuar con partes de la página a medida que estén disponibles en lugar de esperar a que se cargue la página completa. El streaming es especialmente beneficioso para páginas donde la obtención de datos podría introducir demoras, ya que permite la visualización inmediata del contenido disponible que no está bloqueado por la obtención de datos.

Next.js selecciona inteligentemente la estrategia de renderizado óptima para cada ruta, pero los desarrolladores tienen la flexibilidad de ajustar la configuración de caché, los tiempos de revalidación y decidir si utilizar streaming de interfaz de usuario.

Sin embargo, es esencial considerar que la mayoría de las páginas web no encajan perfectamente en categorías puramente estáticas o puramente dinámicas. Aprovechando los mecanismos de caché y las estrategias de renderizado de Next.js, puedes ajustar el equilibrio entre servir contenido estático y respuestas generadas dinámicamente.

-----

## Implementing Server Components

Con una comprensión sólida del renderizado del lado del servidor y las estrategias de renderizado de los Componentes de Servidor, estamos listos para profundizar en cómo implementar Componentes de Servidor de manera efectiva en los próximos ejercicios.

Hasta ahora, has visto Componentes de Servidor en acción para el renderizado de interfaces. La simplicidad de usar Componentes de Servidor proviene del diseño de Next.js: no se necesita configuración adicional. Creas un componente React y Next.js se encarga de orquestar el renderizado del lado del servidor.

Aquí tienes un ejemplo de cómo se ve un Componente de Servidor:

```tsx
// UserProfile.tsx
import React from 'react';

export default function UserProfile({ userId }: { userId: string }) {
  const userData = fetchUserData(userId);
  return (
    <div>
      <h1>Viendo a {userData.firstName}</h1>
      <p>Nombre: {userData.fullName}</p>
      <p>Contacto: {userData.email}</p>
    </div>
  );
}
```

Este ejemplo muestra un Componente de Servidor obteniendo y mostrando datos de usuario. A diferencia de los componentes React típicos, los Componentes de Servidor están diseñados para operaciones del lado del servidor como esta: renderizar contenido basado en lógica de servidor sin involucrar interactividad del lado del cliente.

Los Componentes de Servidor difieren de los componentes React tradicionales — ahora conocidos como Componentes Cliente en Next.js — en varios aspectos clave:

- Se ejecutan en el servidor, utilizando recursos del servidor para tareas como renderizar contenido.
- No contribuyen al bundle de JavaScript del lado del cliente, aligerando la carga en el cliente.
- Son capaces de obtener datos en el servidor y enviar solo el resultado necesario al cliente.

Un consejo práctico al trabajar con Componentes de Servidor es considerar dónde aparecen los mensajes de `console.log()`. Los logs de los Componentes de Servidor aparecerán en la terminal del servidor, no en la consola del navegador. Esta distinción es un recordatorio útil de la división servidor-cliente, indicando dónde se está ejecutando tu código.

**¿Dónde aparecen los logs de los Componentes Cliente?**
Los logs de los Componentes Cliente pueden aparecer tanto en la terminal del servidor como en la consola del navegador, ya que, por defecto, se pre-renderizan en el servidor para generar el HTML inicial.

Al integrar Componentes de Servidor, considera el entorno de cada componente — servidor, cliente, o ambos — para asegurar un rendimiento eficiente mediante una distribución adecuada del renderizado y la carga computacional.

En el editor de código, hay una implementación mínima de una aplicación web de bandeja de entrada de correo electrónico. `<HomePage>` es un Componente de Servidor que obtendrá los correos electrónicos mediante la función `fetchEmailData()` y se los pasará al componente `<EmailFilter>` para mostrar el contenido. Practicaremos la implementación de Componentes de Servidor con esta aplicación web de correo electrónico.

----

## Using Client Components with Server Components

Centrémonos en usar Componentes Cliente y Componentes de Servidor juntos — comprender el patrón de composición nos permitirá aprovechar las fortalezas del servidor y del cliente.

Hemos utilizado la directiva `'use client'` en lecciones anteriores para definir componentes que requieren interacciones del usuario. Esta directiva es crucial para delimitar el entorno de ejecución de los componentes.

Considera un escenario en una aplicación Next.js donde tanto contenido estático como dinámico coexisten en la misma vista. Necesitamos asegurarnos de que cada parte se delegue al entorno de ejecución adecuado: el servidor manejando contenido estático, como perfiles de usuario, y el cliente gestionando contenido interactivo, como widgets, cuando corresponda. Colocar un Componente Cliente dentro de un Componente de Servidor es sencillo; una importación con uso directo es un patrón válido.

```tsx
// ServerComponent.tsx
import ClientComponent from './ClientComponent';
export default function ServerComponent() {
  return (
    <div>
      <h1>Mi Componente de Servidor</h1>
      <ClientComponent />
    </div>
  );
}
```

Los componentes pueden anidarse en ambas direcciones. Los Componentes de Servidor también pueden anidarse dentro de Componentes Cliente. Este patrón es más complicado — para pasar un Componente de Servidor a un Componente Cliente, debemos pasarlo como prop. Una forma común de hacerlo es a través de la prop `children`.

El siguiente ejemplo demuestra cómo usar un Componente de Servidor en un Componente Cliente:

```tsx
// ClientComponent.tsx
'use client'
 
import { useState } from 'react';

export default function ClientComponent({ children }: { children: React.ReactNode }) {
  const [state, toggleState] = useState(true);
  return <div> {children} </div>
}

// Page.tsx
import ClientComponent from './ClientComponent.tsx'
import ServerComponent from './ServerComponent.tsx'

export default function Page() {
  return (
    <ClientComponent>
      <ServerComponent />
    </ClientComponent>
  )
}
```

En este ejemplo, `ServerComponent` se pasa a `ClientComponent` como hijo. Cuando llega una solicitud, el servidor procesa todos los Componentes de Servidor primero, incluyendo aquellos dentro de los Componentes Cliente. Luego, el servidor envía un payload que contiene instrucciones para organizar estos componentes en el lado del cliente. React en el lado del cliente utiliza este payload para combinar los Componentes de Servidor y Cliente en una estructura coherente con la que el usuario puede interactuar.

Este enfoque nos permite utilizar un enfoque de renderizado híbrido.

Cuando elegimos manejar componentes de esta manera, es esencial considerar el patrón de composición. Dado que los Componentes de Servidor se renderizan antes que los Componentes Cliente, no se puede incrustar un Componente de Servidor dentro de un Componente Cliente sin causar una solicitud innecesaria al servidor.

Sin pasarlo como prop, el Componente de Servidor adoptaría el entorno de renderizado de su padre y se renderizaría del lado del cliente. En cambio, si pasamos el Componente de Servidor a través de una prop a un Componente Cliente — o, específicamente, a través de la prop `children` como se demuestra en el ejemplo — pueden desacoplarse y renderizarse de forma independiente, evitando que código destinado al servidor se cuele en el cliente.

Utilizando los patrones de composición de Componentes de Servidor y Componentes Cliente, actualizarás la aplicación web con un filtro de categorías para que los usuarios puedan filtrar y ver solo los correos electrónicos de la categoría seleccionada.

----

## Fallback Components with React Suspense

Continuando con la implementación efectiva de Componentes de Servidor, analizaremos React Suspense y los Componentes Fallback para asegurar que los usuarios encuentren una interfaz de usuario receptiva y atractiva, incluso mientras el servidor realiza el trabajo pesado de obtener o renderizar componentes.

Dado que los Componentes de Servidor se ejecutan en el servidor, los usuarios pueden experimentar breves esperas mientras el servidor renderiza y entrega los componentes. Esta pausa puede resultar en pantallas en blanco temporales o interfaces no responsivas. En su lugar, podemos usar Componentes Fallback dentro de React Suspense como marcadores de posición interinos que señalan actividad a través de cargadores, pantallas esqueleto o marcadores de posición personalizados durante la carga.

React Suspense es una característica incorporada de React que permite "suspender" el renderizado de un componente mientras se espera alguna operación asíncrona, como la obtención de datos o la división de código. Funciona capturando promesas lanzadas por componentes que esperan datos asíncronos, y gestiona el renderizado del contenido de respaldo hasta que los datos estén listos.

Introduce un Límite de Suspense (Suspense Boundary), un patrón poderoso para estructurar la interfaz de usuario de tu aplicación y manejar los estados de carga de manera elegante. Este límite no solo indica la actividad continua del lado del servidor a través de un marcador de posición visual, sino que también asegura que, al completarse la acción, el contenido real se muestre de manera fluida, definiendo el alcance dentro del cual React debe mostrar el contenido de respaldo.

Ya hemos visto esto antes con el archivo reservado `loading.tsx`, que sirve como indicador de carga pero envuelve toda la página. En cambio, podemos crear manualmente Límites de Suspense, lo que nos da control total sobre qué componentes requieren un estado de carga y gestionar esa lógica nosotros mismos.

Next.js mejora este mecanismo con sus capacidades de streaming, permitiendo que los componentes dentro de un Límite de Suspense se transmitan al cliente a medida que se renderizan en el servidor. Este proceso de streaming, junto con Suspense, significa que los usuarios pueden comenzar a interactuar con las partes de la página que están listas mientras otras aún se están preparando en el servidor.

Aquí hay un ejemplo de cómo se vería esto en Next.js:

```tsx
import { Suspense } from 'react'
import UtilityBill from './UtilityBill.tsx'

const FallbackComponent = () => <p>Cargando detalles de la factura...</p>

export default function BillingPage() {
  return (
    <main>
      <h1>Tu Factura de Servicios</h1>
      <Suspense fallback={<FallbackComponent />}>
        <UtilityBill billType="electric" userId="codey24" />
      </Suspense>
    </main>
  );
}
```

Analicémoslo:

- Se define un componente `<FallbackComponent>` con un indicador de carga.
- Un `<Suspense>` envuelve al componente `<UtilityBill>`.
- Se especifica `<FallbackComponent>` como interfaz de usuario de respaldo mientras `<UtilityBill>` se carga.

Al colocar tus Límites de Suspense, considera las siguientes preguntas:

- Considerando la experiencia del usuario, ¿en qué aspectos específicos del streaming de la página te enfocarías?
- En cuanto a la priorización de contenido, ¿qué partes de la página considerarías más cruciales para cargar primero?
- Dado que algunos componentes requieren obtener datos, ¿cómo estructurarías las dependencias entre componentes y fuentes de datos?

Para practicar nuestra comprensión de los Componentes Fallback y los Límites de Suspense, volveremos a ver la aplicación web de correo electrónico con el objetivo de implementar un Límite de Suspense y usar Componentes Fallback.

En los siguientes puntos de control, la aplicación web de correo electrónico ha recibido algunas actualizaciones. La función `fetchEmailData()` en `utils.tsx` ahora es asíncrona y simula latencia de red con un retraso. El componente `<EmailFilter>` se ha actualizado para volverse asíncrono y manejar adecuadamente la Promesa devuelta por `fetchEmailData()`.

----

## Client vs Server Components

A medida que finalizamos nuestra inmersión en los Componentes de Servidor, destilemos claramente nuestra comprensión de los Componentes Cliente y de Servidor.

Los **Componentes de Servidor** son la columna vertebral de la eficiencia de Next.js; úsalos cuando:

- **Obtengas datos**: el componente está inherentemente "más cerca" de la fuente de datos y puede acceder a ellos con menor latencia que la que podría existir en una operación de obtención de datos del lado del cliente.
- **Accedas a recursos del backend**: el componente puede interactuar con recursos del backend de forma segura sin exponer las operaciones al cliente.
- **Asegures información sensible**: el componente puede mantener datos como tokens de acceso y claves API en el servidor.
- **Optimices el rendimiento**: al gestionar grandes dependencias en el servidor, el componente reduce el tamaño del paquete JavaScript del lado del cliente.

Los **Componentes Cliente** son responsables de la interactividad de tu aplicación; úsalos cuando:

- **Añadas interactividad**: hacen que tu aplicación responda a las entradas e interacciones del usuario.
- **Gestiones estado y efectos**: permiten que las aplicaciones tengan estado y efectos secundarios con React Hooks.
- **Utilices APIs del navegador**: los Componentes Cliente nos permiten usar características específicas del navegador como geolocalización o localStorage.
- **Añadas hooks personalizados**: aplica hooks personalizados de React para gestionar lógica con estado.

Integrar ambos tipos de componentes de manera reflexiva es esencial para aprovechar las fortalezas del procesamiento del lado del servidor mientras se mantiene una interfaz de usuario dinámica e interactiva.

Comprender los matices de los Componentes Cliente y de Servidor es crucial para construir aplicaciones Next.js eficientes que equilibren la optimización del rendimiento y la experiencia del usuario.

En los siguientes puntos de control, implementaremos una aplicación simple de geolocalización que verifica si el usuario está cerca de una ciudad importante. Esta aplicación utilizará tanto Componentes de Servidor como Componentes Cliente.

----

## Review

Hemos llegado al final de nuestra lección sobre Componentes de Servidor.

Hagamos un repaso:

- Los **Componentes de Servidor**, conocidos como React Server Components (RSCs), son el tipo de componente predeterminado en Next.js que se renderizan en el servidor.
- El **renderizado de los Componentes de Servidor** implica una coordinación entre servidor y cliente para la distribución de tareas y la actualización del DOM.
- Existen **tres estrategias de renderizado** para los Componentes de Servidor. Estas estrategias proporcionan una distribución eficiente del contenido, generación de contenido en tiempo real y tiempos de carga mejorados.
- La **estrategia de renderizado estático** es ideal para contenido estático, ya que pre-genera páginas para una distribución eficiente a través de CDN.
- La **estrategia de renderizado dinámico** es la mejor para contenido personalizado o sensible al tiempo, ya que genera contenido en tiempo real por solicitud, aunque puede ralentizar los tiempos de respuesta.
- La **estrategia de streaming** funciona entregando fragmentos de interfaz de usuario de forma progresiva, permitiendo a los usuarios interactuar con partes de la página a medida que se cargan.
- Un **Componente de Servidor se puede implementar** creando un componente React y dejando que Next.js gestione el renderizado del lado del servidor.
- Los Componentes de Servidor funcionan en el **lado del servidor**, reduciendo la carga del lado del cliente al evitar contribuir al paquete de JavaScript y optimizando el rendimiento de la aplicación al obtener datos en el servidor.
- **React Suspense y los Componentes Fallback** permiten a los desarrolladores crear Límites de Suspense personalizados, ofreciendo retroalimentación informativa a los usuarios durante demoras de carga.
- Las aplicaciones Next.js pueden usar **Componentes Cliente y Componentes de Servidor juntos** en un patrón de composición, creando un enfoque de renderizado híbrido.
- Los Componentes de Servidor **no deben importarse directamente** en módulos de Componentes Cliente, sino pasarse a través de una prop para un renderizado independiente.
- Los **Componentes de Servidor y Componentes Cliente tienen cada uno sus roles**: los Componentes de Servidor manejan interacciones con datos y backend de manera eficiente, mientras que los Componentes Cliente potencian la interactividad y gestionan el estado.

Ahora que tienes un conocimiento sólido de los Componentes de Servidor, puedes optimizar las interacciones del lado del servidor dentro de los componentes, simplificando el código y reduciendo la carga mental al trabajar con datos dinámicos.

-----