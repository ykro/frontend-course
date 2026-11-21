---
name: next-componentes-servidor-y-cliente
description: Usar cuando una app de Next.js con App Router carga lento o tiene un bundle de JavaScript grande, hay 'use client' en páginas o layouts enteros, se piden datos con useEffect en una página, o hay que decidir qué va en un Server Component y qué en un Client Component.
---

# Componentes de servidor y de cliente en Next.js

En el App Router todo es Server Component hasta que se marca `'use client'`. Lo que corre en el servidor no se descarga al navegador. La regla es mantener la página en el servidor, pedir los datos ahí y bajar `'use client'` a la hoja más pequeña que necesita estado, eventos o APIs del navegador.

1. Mide antes. Si el build no imprime tamaños por ruta (Next 16 con Turbopack), usa `next analyze` (en versiones anteriores, `next experimental-analyze`, desde 16.1) o mide en Chrome DevTools, Network, filtro JS, con la caché desactivada, el JavaScript que descarga la ruta. Sin ese número no se puede decir que mejoró.
2. Lee la versión de Next en `package.json` y su documentación en `node_modules/next/dist/docs/` antes de afirmar APIs (por ejemplo, en Next 15 y 16 `params` y `searchParams` son promesas).
3. Por cada archivo con `'use client'`, pregunta qué necesita del navegador (estado, efectos, eventos, `window`). Si la respuesta es "nada" o "solo una parte", quita la directiva y extrae esa parte a un componente cliente pequeño.
4. Pide los datos en el servidor. La página es `async` y llama a la función de datos del proyecto (búscala; si no existe, créala en `lib/` y dilo). No llames a tu propia ruta `/api` desde un Server Component; llama a la función directamente.
5. Saca del cliente las librerías pesadas. Fechas con `Intl.DateTimeFormat`, ordenar con `toSorted`, gráficas con `next/dynamic` y `ssr: false` solo donde se ven (`ssr: false` solo funciona dentro de un Client Component; en un Server Component da error). Cada librería que quites, confírmala en el bundle.
6. Usa `next/image` con `width`, `height` (o `fill` y `sizes`) y `preload` en la imagen principal (`priority` antes de Next 16, que lo marca obsoleto), y `next/font` para las fuentes.
7. Decide la frescura con el dueño del producto. Si la página puede mostrar datos de hace unos minutos, se cachea; si no, es dinámica. No pongas `revalidate = 300` sin preguntar (skill `next-cache-y-revalidacion`).
8. No dejes archivos de ejemplo ni imports a módulos que no existen. Corre `npm run typecheck` y `npm run build`, y reporta el tamaño antes y después.

El ejemplo muestra la forma; no agregues componentes ni funciones que el original no tenía sin decirlo.

```tsx
// app/page.tsx (Server Component)
import Image from 'next/image'
import { listProducts } from '@/lib/products'
import { ProductSearch } from './product-search' // 'use client', solo el buscador

export default async function Home() {
  const products = (await listProducts()).toSorted((a, b) => a.name.localeCompare(b.name))
  return (
    <main>
      <Image src="/banner.webp" alt="Ofertas de la semana" width={1200} height={400} preload sizes="100vw" />
      <h1>Tienda {new Date().getFullYear()}</h1>
      <ProductSearch products={products} />
    </main>
  )
}
```

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
