# React Hook Form y Zod

## El problema con los formularios controlados a mano

En las dos lecciones anteriores vimos cómo construir formularios con componentes controlados (una variable de estado por campo, actualizada en cada `onChange`) y con componentes no controlados (leyendo el valor directamente del DOM a través de una `ref`). Ambos enfoques funcionan bien para formularios chicos, pero a medida que un formulario crece en cantidad de campos y en reglas de validación, escribirlo a mano empieza a mostrar sus costos: un `useState` por campo, un `handleChange` que se repite con pequeñas variaciones, y una función de validación que crece hasta convertirse en un bloque de `if` difícil de mantener, donde las reglas de negocio (qué es un email válido, qué rango de edad es aceptable) quedan mezcladas con la lógica de renderizado.

**React Hook Form** (RHF) y **Zod** son dos librerías que, combinadas, resuelven cada una una mitad de este problema: RHF se encarga de gestionar el **estado del formulario**, y Zod se encarga de las **reglas de validación**. Ninguna de las dos depende de la otra — cada una puede usarse por separado — pero juntas son la combinación más común en el ecosistema de React para formularios de producción.

-----

## React Hook Form: el estado del formulario sin re-renders de sobra

React Hook Form gestiona los valores, el estado de "tocado" (`touched`), el estado de envío (`isSubmitting`) y los errores de cada campo, sin que vos tengas que declarar un `useState` por cada uno. La diferencia clave frente a un formulario controlado a mano es que RHF es, por defecto, **no controlado**: en lugar de escuchar cada `onChange` y actualizar estado (lo que dispara un re-render del componente en cada tecla presionada), usa `refs` internamente para leer los valores de los inputs solo cuando hace falta —típicamente, al enviar el formulario—, evitando re-renders innecesarios.

El punto de entrada es el hook `useForm()`:

```tsx
import { useForm } from 'react-hook-form';

type ContactFormValues = {
  name: string;
  email: string;
};

function ContactForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<ContactFormValues>();

  const onSubmit = (data: ContactFormValues) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} />
      {errors.name && <span>{errors.name.message}</span>}

      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}

      <button type="submit">Enviar</button>
    </form>
  );
}
```

Tres piezas hacen todo el trabajo:

* **`register('name')`** conecta un input al formulario. Devuelve un objeto con `name`, `onChange`, `onBlur` y `ref`, que se esparce (`{...register('name')}`) directamente sobre el `<input>`. A partir de ahí, RHF sabe leer y validar ese campo sin que vos manejes su estado manualmente.
* **`handleSubmit(onSubmit)`** envuelve tu función de envío: primero corre las validaciones, y solo si todas pasan, llama a `onSubmit` con los valores ya tipados y listos para usar.
* **`formState.errors`** contiene los errores de validación de cada campo, indexados por nombre, para que puedas mostrarlos junto al input correspondiente.

-----

## Zod: el esquema como contrato

Sin una librería de validación, las reglas de un formulario ("el email tiene que tener formato válido", "la edad tiene que ser mayor a 18") suelen terminar dispersas: una parte en el `onChange` de un input, otra en una función `validate()` separada, y a veces una tercera copia de la misma regla en el backend. **Zod** propone centralizar todas esas reglas en un único **esquema**, declarado de forma explícita, que funciona como un contrato: describe exactamente qué forma tienen los datos válidos, en un solo lugar.

```tsx
import { z } from 'zod';

const contactSchema = z.object({
  name: z.string().min(1, 'El nombre es obligatorio'),
  email: z.string().email('Ingresá un email válido'),
});
```

Este mismo esquema cumple dos roles a la vez: sirve para **validar** datos en tiempo de ejecución (rechazando lo que no cumpla las reglas) y, gracias a la utilidad `z.infer`, sirve para **derivar el tipo de TypeScript** correspondiente, sin tener que escribir el `type` por separado y arriesgarte a que el tipo y la validación se desincronicen con el tiempo:

```tsx
type ContactFormValues = z.infer<typeof contactSchema>;
// equivalente a: type ContactFormValues = { name: string; email: string };
```

-----

## Conectando ambas piezas: zodResolver

RHF no sabe nada de Zod por defecto — necesita un **resolver** que traduzca el resultado de validar contra un esquema de Zod al formato de errores que RHF espera. Ese puente lo provee el paquete `@hookform/resolvers`:

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const contactSchema = z.object({
  name: z.string().min(1, 'El nombre es obligatorio'),
  email: z.string().email('Ingresá un email válido'),
});

type ContactFormValues = z.infer<typeof contactSchema>;

function ContactForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<ContactFormValues>({
    resolver: zodResolver(contactSchema),
  });

  const onSubmit = (data: ContactFormValues) => {
    // Acá "data" ya está validado Y tipado según contactSchema.
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} placeholder="Nombre" />
      {errors.name && <span>{errors.name.message}</span>}

      <input {...register('email')} placeholder="Email" />
      {errors.email && <span>{errors.email.message}</span>}

      <button type="submit">Enviar</button>
    </form>
  );
}
```

A partir de acá, `useForm<ContactFormValues>` conoce exactamente qué campos existen y de qué tipo son (gracias al genérico derivado del esquema), y `handleSubmit` no llama a `onSubmit` hasta que los datos pasan la validación de Zod. Las reglas de validación viven en un único lugar —el esquema—, en vez de estar repetidas entre el `onChange` de cada input y una función de validación aparte.

-----

## Reglas adicionales por campo

Zod permite encadenar tantas reglas como necesite cada campo, más allá de lo básico (`min`, `email`):

```tsx
const registrationSchema = z.object({
  username: z
    .string()
    .min(3, 'Mínimo 3 caracteres')
    .max(20, 'Máximo 20 caracteres')
    .regex(/^[a-zA-Z0-9_]+$/, 'Solo letras, números y guion bajo'),
  age: z
    .number()
    .int('Tiene que ser un número entero')
    .min(18, 'Tenés que ser mayor de edad'),
  password: z.string().min(8, 'Mínimo 8 caracteres'),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: 'Las contraseñas no coinciden',
  path: ['confirmPassword'], // el error se asocia a este campo específico
});
```

El método `.refine()` es el que permite expresar reglas que dependen de **más de un campo a la vez** (como confirmar que dos contraseñas coincidan), algo que sería incómodo de expresar validando cada campo por separado. El segundo argumento de `.refine()` — el mensaje y el `path` — le indica a Zod (y, en cadena, a RHF) a qué campo específico asociar el error, para poder mostrarlo junto al input correspondiente en lugar de como un error genérico del formulario completo.

-----

## Componentes que no son un `<input>` nativo: Controller

`register()` funciona espesando sus props (`ref`, `onChange`, `onBlur`) directamente sobre un elemento nativo como `<input>` o `<select>`. El problema aparece con componentes de librerías de terceros —un selector de fecha (*datetime picker*), un combobox con autocompletado, un slider— que no exponen exactamente esa misma interfaz, o que gestionan su valor de una forma distinta a un input nativo. Para esos casos, React Hook Form expone el componente `<Controller>`, que actúa de intermediario entre RHF y cualquier componente controlado:

```tsx
import { Controller, useForm } from 'react-hook-form';
import DatePicker from 'algun-datepicker-de-terceros';

function EventForm() {
  const { control, handleSubmit } = useForm<{ eventDate: Date }>();

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      <Controller
        name="eventDate"
        control={control}
        render={({ field }) => (
          <DatePicker selected={field.value} onChange={field.onChange} />
        )}
      />
      <button type="submit">Guardar</button>
    </form>
  );
}
```

`Controller` le pasa a la función `render` un objeto `field` (con `value`, `onChange`, `onBlur`, etc.) que vos adaptás manualmente a la API específica del componente de terceros — a diferencia de `register()`, que asume que el componente destino ya habla el mismo idioma que un `<input>` nativo.

-----

## Mensajes de error e internacionalización

Los mensajes de error de un esquema de Zod son strings comunes, lo que significa que se pueden combinar sin fricción con una librería de internacionalización (i18n): en lugar de escribir el mensaje literal dentro del `.min()` o `.email()`, se puede llamar a la función de traducción de la librería que estés usando (por ejemplo, `t('validation.emailInvalid')`) y pasar el resultado como mensaje. De esa forma, el esquema de validación queda desacoplado del idioma en el que finalmente se muestra cada error, y agregar un nuevo idioma no requiere tocar la lógica de validación en absoluto.

-----

## Resumen

* **React Hook Form** gestiona el estado del formulario (valores, errores, envío) usando `refs` en lugar de `useState` por campo, lo que evita re-renders innecesarios en cada tecla presionada.
* **Zod** centraliza las reglas de validación en un **esquema**, que funciona como un contrato único tanto para validar datos en tiempo de ejecución como para derivar el tipo de TypeScript correspondiente con `z.infer`.
* **`zodResolver`** (de `@hookform/resolvers/zod`) conecta ambas librerías, pasado como opción `resolver` a `useForm()`.
* **`.refine()`** permite expresar reglas que dependen de varios campos a la vez, como confirmar que dos contraseñas coincidan.
* **`<Controller>`** es el puente necesario para conectar RHF con componentes de terceros (date pickers, selects custom) que no exponen la misma interfaz que un `<input>` nativo.
* Como los mensajes de error son strings comunes, se combinan sin problema con una librería de internacionalización.
