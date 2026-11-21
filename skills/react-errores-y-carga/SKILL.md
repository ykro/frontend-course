---
name: react-errores-y-carga
description: Usar cuando una página de React o Next.js se queda en blanco o se rompe si falla una petición, no hay estados de carga, error, vacío o no encontrado, o hay que agregar error.tsx, global-error.tsx, loading.tsx, not-found.tsx, Suspense o error boundaries.
---

# Estados de error, carga, vacío y no encontrado

Cada pantalla que depende de datos tiene cuatro estados además del feliz. Cargando, error con forma de reintentar, vacío y no encontrado. En el App Router cada uno tiene su archivo; en React sin framework, Suspense y un error boundary. Ninguno muestra trazas ni datos internos.

1. Lista las rutas que piden datos y, para cada una, qué pasa hoy si la petición tarda, falla, devuelve vacío o el id no existe. Pruébalo forzando cada caso (una variable de entorno o un doble que falla, un id inventado).
2. Agrega por segmento lo que falte.

   | Archivo | Para qué |
   |---|---|
   | `loading.tsx` | esqueleto con la forma de la página mientras carga; usa Suspense por debajo |
   | `error.tsx` (`'use client'`) | mensaje humano y botón Reintentar; recibe `error` (con `digest`) y `retry` (o `reset` en versiones anteriores) |
   | `not-found.tsx` y `notFound()` | cuando el recurso no existe; la página llama `notFound()` en lugar de lanzar un error |
   | `app/global-error.tsx` | errores en el layout raíz; necesita sus propias etiquetas `html` y `body` |

3. Revisa en `node_modules/next/dist/docs/01-app/03-api-reference/03-file-conventions/error.md` las props exactas de tu versión antes de escribir `error.tsx`.
4. Un `loading.tsx` de un segmento padre también se muestra en sus rutas hijas; si la forma no corresponde, pon uno propio en la hija. `generateMetadata` también debe tolerar el fallo. Con streaming, el error llega con HTTP 200; se comprueba en el navegador, no con el código de estado. Para partes independientes de una página (comentarios, recomendaciones), envuélvelas en su propio `<Suspense fallback>` para que una lenta no bloquee el resto.
5. Registra el error en el servidor con su `digest` y muestra al usuario solo un mensaje y la acción siguiente. Nunca `error.message` crudo de la base de datos.
6. Distingue errores esperados de inesperados. Que un evento esté lleno o un formulario sea inválido es un resultado que la acción devuelve y la UI muestra; `error.tsx` es para lo inesperado.
7. Cubre también las acciones del usuario en esa pantalla (inscribirse, guardar), no solo la carga.
8. Prueba cada estado. Un test por estado con el doble que falla o devuelve vacío, y una comprobación manual en `npm run build && npm start` forzando el error.

```tsx
'use client'
// Next 16.3 o posterior. En versiones anteriores la prop se llama reset (o unstable_retry en 16.2).
export default function EventError({ error, retry }: { error: Error & { digest?: string }; retry: () => void }) {
  return (
    <section role="alert">
      <h2>No pudimos cargar este evento</h2>
      <p>Intenta de nuevo en un momento. Si sigue fallando, comparte este código con soporte: {error.digest}</p>
      <button type="button" onClick={() => retry()}>Reintentar</button>
    </section>
  )
}
```

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
