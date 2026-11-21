---
name: next-server-actions-seguras
description: Usar cuando hay que escribir o revisar una Server Action de Next.js ('use server', formularios con action, useActionState, revalidatePath, revalidateTag, updateTag) que crea, cambia o borra datos, o cuando una acción recibe ids o usuarios desde el formulario.
---

# Server Actions seguras en Next.js

Una Server Action es un endpoint público. Cualquiera puede llamarla con los datos que quiera, aunque el botón esté escondido. Por eso cada acción valida la entrada, sabe quién llama sin creerle al formulario, comprueba que puede tocar ese recurso y solo después cambia datos y revalida.

1. Lee la versión de Next en `package.json` y la documentación que trae (`node_modules/next/dist/docs/`) antes de usar APIs de caché o revalidación; cambian entre versiones.
2. Separa la lógica en una función pura o un servicio que recibe dependencias (repositorio, reloj, usuario) y es fácil de probar. La acción solo lee el `FormData`, valida, obtiene la identidad, llama al servicio y revalida.
3. Valida con Zod (`safeParse`) antes de leer secretos o tocar datos, y devuelve errores por campo con una forma estable para `useActionState` (si el proyecto no tiene una, algo como `{ ok: false, fieldErrors }`). Un id del formulario con formato inválido se responde igual que uno que no existe.
4. La identidad sale de la sesión del servidor (cookie firmada, proveedor de auth), nunca de un campo `userId` del formulario. Si la app no tiene sesión y un código autoriza algo (cancelar, entrar), ese código (boleto, enlace mágico) debe ser largo y aleatorio (128 bits o más) o firmado con HMAC completo, comparado con `timingSafeEqual` y con límite de intentos. Un comprobante que solo se muestra no autoriza nada; si su formato lo fija un test, no lo cambies y anótalo como riesgo si algún día se usa para autorizar.
5. Autoriza sobre el recurso. Carga el registro y comprueba que pertenece a quien llama antes de cambiarlo; la respuesta de "no existe" y "no es tuyo" puede ser la misma para no revelar ids.
6. Maneja los estados del dominio (ya cancelada, cupo lleno) como resultados esperados con mensaje, no como excepciones genéricas.
7. Revalida lo justo después del éxito, la ruta o la etiqueta que muestra ese dato. No llames `revalidatePath('/')` por costumbre.
8. Usa la forma de resultado y la revalidación que ya use el proyecto; el ejemplo es la forma general. Si la acción necesita un cambio en una interfaz o un doble de pruebas compartido, cambia esa interfaz y su doble en el mismo cambio; si las reglas del repositorio prohíben tocar el doble, pregunta antes. Crear una interfaz paralela para esquivar el test deja dos caminos.
9. Prueba el servicio con sus dependencias falsas (éxito, entrada inválida, recurso ajeno, estado ya final). Si ya hay tests que cubren esos casos o las reglas prohíben tocarlos, no dupliques; escribe los que falten solo donde el repositorio lo permite. Corre el script de verificación del proyecto (`npm run verify` o typecheck, lint, test y build).

```ts
'use server'
import { z } from 'zod'
import { updateTag } from 'next/cache'
import { getSession } from '@/lib/auth'
import { cancelReservation } from '@/lib/reservations'

const input = z.object({ reservationId: z.uuid() })

export async function cancelReservationAction(_prev: unknown, formData: FormData) {
  const parsed = input.safeParse({ reservationId: formData.get('reservationId') })
  if (!parsed.success) return { ok: false as const, error: 'Solicitud no válida' }
  const session = await getSession()
  if (!session) return { ok: false as const, error: 'Inicia sesión' }
  const result = await cancelReservation(parsed.data.reservationId, session.userId)
  if (!result.ok) return result
  updateTag(`event-${result.eventId}`)
  return { ok: true as const }
}
```

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
