---
name: react-formularios-zod
description: Usar cuando hay que construir o corregir un formulario en React con validación (React Hook Form, Zod, reglas que dependen de otro campo, números que llegan como texto, mensajes de error accesibles), o el formulario deja pasar datos inválidos o el servidor recibe algo distinto de lo que validó el cliente.
---

# Formularios con React Hook Form y Zod

Un esquema de Zod es la fuente de verdad. De él salen el tipo del formulario, la validación en el cliente y la validación en el servidor, que se repite siempre porque el cliente se puede saltar. Los errores se anuncian al lector de pantalla y se prueban por rol.

1. Revisa las versiones instaladas en `package.json` (`zod`, `react-hook-form`, `@hookform/resolvers`) antes de escribir. Zod 4 cambió APIs (`z.email()` en lugar de `z.string().email()`, mensajes con `error`), y el resolver necesita una versión compatible.
2. Escribe el esquema en su propio archivo con límites claros. Textos con `trim()`, `min` y `max`; números con `int()`, `min` y `max`. Las reglas que cruzan campos van en `superRefine` y ponen el error en el campo que lo debe mostrar (`path: ['groupName']`).
3. Si un detalle del formulario no está en el pedido (cómo se elige "grupo", qué pasa con el nombre del grupo si se desmarca), pregunta o márcalo como supuesto en tu respuesta.
4. Conecta con `useForm<FormValues>({ resolver: zodResolver(schema) })`. Para mostrar u ocultar campos según otro valor usa `useWatch`, no `watch`. Al desmarcar una opción, limpia los campos que dependían de ella (`setValue("groupName", undefined)`). Los inputs numéricos usan `register('tickets', { valueAsNumber: true })`; los componentes que no son inputs nativos, `Controller`.
5. Cada campo tiene `<label htmlFor>`, `aria-invalid` cuando hay error y `aria-describedby` hacia el mensaje, que vive en un elemento con id. El botón de enviar se desactiva mientras se envía, no mientras hay errores.
6. Al enviar, el servidor (Server Action o API) vuelve a validar con el mismo esquema y devuelve errores por campo. Solo limpia el formulario (`reset()`) si el servidor confirmó el éxito.
7. Prueba el esquema con casos de borde (0, 1, 6 y 7 boletos; grupo sin nombre; nombre con solo espacios) y el componente por rol con Testing Library (escribir, enviar, leer el error con `getByRole` o `getByText` asociado al campo).
8. Si no hay `package.json`, entrégalo como componente y esquema con las dependencias necesarias y pruébalo en un proyecto mínimo. Corre `npm run typecheck`, `npm test` y `npm run lint`. Si no puedes correrlos, dilo como pendiente bloqueante.

```ts
import { z } from 'zod'

export const registrationSchema = z.object({
  name: z.string().trim().min(2, { error: 'Escribe tu nombre' }).max(80),
  email: z.email({ error: 'Correo no válido' }),
  tickets: z.number({ error: 'Escribe cuántos boletos' }).int().min(1).max(6),
  kind: z.enum(['individual', 'grupo']),
  groupName: z.string().trim().max(80).optional(),
}).superRefine((v, ctx) => {
  if (v.kind === 'grupo' && !v.groupName) {
    ctx.addIssue({ code: 'custom', path: ['groupName'], message: 'Escribe el nombre del grupo' })
  }
})

export type RegistrationValues = z.infer<typeof registrationSchema>
```

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
