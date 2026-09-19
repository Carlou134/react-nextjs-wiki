# Composición y Formularios

## Los componentes se arman por partes

Casi todos los componentes interactivos de shadcn/ui no son un único elemento: son una **familia de partes** que se combinan con JSX. Si leíste la lección de Compound Components de los patrones de React, esto te va a resultar familiar, porque es exactamente ese patrón aplicado a una librería real.

El ejemplo más claro es el diálogo (una ventana modal). Se instala con:

```bash
pnpm dlx shadcn@latest add dialog
```

Y se importan todas sus partes de un mismo archivo:

```tsx
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog"
```

Y se usa así:

```tsx
<Dialog>
  <DialogTrigger>Open</DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>¿Estás completamente seguro?</DialogTitle>
      <DialogDescription>
        Esta acción no se puede deshacer. Se eliminará tu cuenta de forma
        permanente y se borrarán tus datos de nuestros servidores.
      </DialogDescription>
    </DialogHeader>
  </DialogContent>
</Dialog>
```

### Leyendo la estructura

Cada parte tiene un rol claro, y entre todas forman una jerarquía que refleja lo que ves en pantalla:

| Parte | Rol |
| --- | --- |
| `Dialog` | El componente raíz: guarda el estado (abierto o cerrado) y coordina a los demás |
| `DialogTrigger` | El elemento que abre el diálogo al hacer clic |
| `DialogContent` | La ventana en sí, que aparece encima del resto de la página |
| `DialogHeader` | El encabezado, que agrupa el título y la descripción |
| `DialogTitle` / `DialogDescription` | El título y el texto explicativo |

Las partes **comparten estado implícitamente**: el `DialogTrigger` sabe abrir el diálogo, y el `DialogContent` sabe si tiene que mostrarse, sin que vos conectes nada a mano. Ese estado compartido viaja por Context, exactamente el mecanismo del patrón Compound Components: un componente padre que es dueño del estado, y partes hijas que lo consumen. Quien usa el diálogo controla la **estructura** (qué va adentro, en qué orden), y las partes se mantienen sincronizadas por debajo.

El resultado es que el componente no necesita una prop por cada personalización: si querés un pie con botones, agregás un `DialogFooter` adentro del `DialogContent`; si querés quitar el encabezado, simplemente no lo ponés.

### Detalles útiles del diálogo

* Trae un **botón de cierre por defecto**; con `showCloseButton={false}` se oculta, o se reemplaza por uno propio.
* Soporta un **pie fijo** (`DialogFooter`) para que las acciones queden visibles mientras el contenido hace scroll.
* La **accesibilidad** (foco atrapado dentro del diálogo, cierre con `Escape`, anuncio en lectores de pantalla) la resuelve la primitiva sobre la que está construido. No la escribiste vos, pero sí tenés que aportar lo que la primitiva no puede adivinar: un título significativo, por ejemplo.

### El trigger con `asChild`

Por defecto, `DialogTrigger` renderiza su propio botón sin estilo. Casi siempre querés usar tu `Button` como disparador, y para eso se usa el `asChild` que vimos en la lección anterior:

```tsx
<Dialog>
  <DialogTrigger asChild>
    <Button variant="outline">Editar perfil</Button>
  </DialogTrigger>
  <DialogContent>
    {/* ... */}
  </DialogContent>
</Dialog>
```

El disparador es **tu** `Button`, con su variante, y `DialogTrigger` le agrega el comportamiento de abrir el diálogo sin agregar un elemento intermedio.

-----

## Formularios

En un formulario real, además de mostrar campos, hay que validarlos, mostrar los errores y comunicar el estado a las tecnologías de asistencia. shadcn/ui no trae su propia lógica de formularios: se apoya en **React Hook Form** y **Zod**, las mismas librerías de la lección de Forms de este wiki, y agrega componentes de presentación para armar cada campo.

### Los componentes de campo

Se agregan con el CLI:

```bash
pnpm dlx shadcn@latest add field input button
```

Y el componente `Field` y sus partes se importan de `@/components/ui/field`. Según la documentación, sirve para componer "campos de formulario accesibles y grupos de inputs", combinando la etiqueta, el control y el texto de ayuda:

```tsx
<FieldGroup>
  <Field>
    <FieldLabel htmlFor="name">Nombre completo</FieldLabel>
    <Input id="name" autoComplete="off" placeholder="Evil Rabbit" />
    <FieldDescription>Esto aparece en las facturas y los emails.</FieldDescription>
  </Field>

  <Field>
    <FieldLabel htmlFor="username">Usuario</FieldLabel>
    <Input id="username" autoComplete="off" aria-invalid />
    <FieldError>Elegí otro nombre de usuario.</FieldError>
  </Field>
</FieldGroup>
```

Cada parte tiene un rol: `FieldGroup` agrupa varios campos, `Field` envuelve un campo individual, `FieldLabel` es la etiqueta (asociada al input por `htmlFor`), `FieldDescription` es el texto de ayuda, y `FieldError` muestra el error. Este mismo esquema de partes es, de nuevo, el patrón de composición.

### Conectarlo con React Hook Form y Zod

La documentación actual conecta esas partes con React Hook Form usando `Controller`, siguiendo tres pasos que ya conocés: definir el esquema con Zod, inicializar el formulario con `useForm` y `zodResolver`, y armar el marcado.

```tsx
import { zodResolver } from "@hookform/resolvers/zod"
import { Controller, useForm } from "react-hook-form"
import * as z from "zod"
import { Button } from "@/components/ui/button"
import { Field, FieldError, FieldLabel } from "@/components/ui/field"
import { Input } from "@/components/ui/input"

const formSchema = z.object({
  title: z.string().min(5, "Debe tener al menos 5 caracteres."),
})

type FormValues = z.infer<typeof formSchema>

export function TitleForm() {
  const form = useForm<FormValues>({
    resolver: zodResolver(formSchema),
    defaultValues: { title: "" },
  })

  function onSubmit(data: FormValues) {
    console.log(data)
  }

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <Controller
        name="title"
        control={form.control}
        render={({ field, fieldState }) => (
          <Field data-invalid={fieldState.invalid}>
            <FieldLabel htmlFor="title">Título</FieldLabel>
            <Input
              {...field}
              id="title"
              aria-invalid={fieldState.invalid}
            />
            {fieldState.invalid && <FieldError errors={[fieldState.error]} />}
          </Field>
        )}
      />
      <Button type="submit">Guardar</Button>
    </form>
  )
}
```

### Qué está pasando ahí

* El **esquema** (`formSchema`) es el contrato de validación, y de él se **deriva el tipo** (`z.infer`), para no escribirlo dos veces.
* `Controller` conecta un campo con el formulario y le entrega a la función `render` dos objetos: `field` (con `value`, `onChange`, etc., que se esparcen sobre el `Input`) y **`fieldState`**, que dice si ese campo es inválido y cuál es su error.
* Ese `fieldState` es lo que alimenta la parte de presentación: `data-invalid` en el `Field` (para dar estilo de error), `aria-invalid` en el `Input` (para comunicar el estado a los lectores de pantalla), y el `FieldError` que muestra el mensaje **solo cuando corresponde**.

Es decir, la validación la hace Zod, el estado del formulario lo maneja React Hook Form, la accesibilidad la aportan los atributos `aria-*`, y shadcn/ui pone la **presentación** consistente con el resto del sistema de diseño. Cada librería hace una sola cosa.

La documentación enfatiza que este enfoque da "flexibilidad completa sobre el marcado y los estilos": en lugar de un componente de formulario cerrado que genera los campos por vos, componés cada campo manualmente. Cuesta más líneas por campo, pero no hay ninguna restricción de diseño.

> **Ejercicio relacionado:** este mismo formulario es una versión con presentación de shadcn/ui del ejercicio de Zod de los ejercicios de práctica, y sirve como buen punto de partida para el ejercicio de formulario genérico.

-----

## Resumen

* Los componentes de shadcn/ui son **familias de partes** (`Dialog`, `DialogTrigger`, `DialogContent`...) que se componen con JSX: es el patrón **Compound Components**, con el estado compartido por Context.
* El componente no necesita una prop por cada personalización: componés la estructura que necesitás.
* `asChild` en un trigger hace que **tu componente** (por ejemplo, un `Button`) sea el disparador, sin un elemento intermedio.
* La **accesibilidad** de comportamiento (foco, teclado) la aportan las primitivas; los **textos** (títulos, etiquetas) los aportás vos.
* Los formularios se construyen sobre **React Hook Form y Zod**: `Controller` entrega `fieldState`, y `Field`, `FieldLabel` y `FieldError` se ocupan de la presentación y de los atributos de accesibilidad.
