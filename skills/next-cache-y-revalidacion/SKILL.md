---
name: next-cache-y-revalidacion
description: Usar cuando una app de Next.js muestra datos viejos en producción pero no en local, un cambio no aparece hasta recargar o esperar, hay que decidir entre página estática, dinámica o cacheada, o se usan 'use cache', cacheLife, cacheTag, revalidatePath, revalidateTag, updateTag o refresh.
---

# Caché y revalidación en Next.js

En `next dev` casi nada se cachea; en producción sí. Por eso "en local se ve y en producción no" suele ser caché, pero no siempre. En Vercel cada función es una instancia sin memoria compartida, así que datos guardados en una variable del servidor también se ven "viejos" o distintos según la instancia. Primero se demuestra cuál de las dos es.

1. Lee la versión de Next en `package.json` y la documentación que trae en `node_modules/next/dist/docs/` (busca `revalidateTag`, `updateTag`, `cacheComponents`). Las APIs de caché cambiaron mucho entre la 14, la 15 y la 16; no las afirmes de memoria.
2. Reproduce en modo producción local. `npm run build && npm start`, crea el dato y mira la lista. Revisa en la salida del build si la ruta es estática o dinámica.
3. Descarta datos en memoria. Con un solo proceso no se ve; levanta dos (`npm start -- -p 3001` y `-p 3002`), escribe en uno y lee en el otro. Busca arreglos o `Map` a nivel de módulo usados como base de datos. Si existen, en Vercel cada instancia tiene su copia y se pierden al reciclarse; la solución es una base de datos real, no revalidar.
4. Si es caché, ubica qué se cachea. La página completa (estática), un `fetch` cacheado, una función con `'use cache'` o `unstable_cache`, o el router del cliente.
5. Revalida en el lugar correcto, después de la escritura y desde el servidor.

   | Necesidad | Herramienta (Next 16) |
   |---|---|
   | Quien hizo el cambio lo ve de inmediato | `updateTag('events')` en la Server Action |
   | Los demás lo ven pronto, se acepta un dato viejo un momento | `revalidateTag('events', 'max')` |
   | Una ruta concreta | `revalidatePath('/events')` |
   | Recargar los datos de la vista actual en el cliente | `refresh()` en la acción o `router.refresh()` |

6. Marca los datos con etiquetas donde se leen (`cacheTag('events')` dentro de `'use cache'`, que en Next 16 requiere `cacheComponents: true` en `next.config`, o `next: { tags: ['events'] }` en `fetch`) para poder invalidarlos sin tocar rutas.
7. Decide la frescura con el dueño del producto. Cuánto puede tardar en verse un cambio y para quién. Con esa respuesta eliges estático con revalidación, cacheado con etiqueta o dinámico.
8. Comprueba en producción local otra vez (crear y ver la lista sin recargar) y deja una prueba de la acción que verifica que llama a la revalidación correcta.
9. Responde primero la causa en dos líneas y después el arreglo. Si el arreglo de fondo tarda (migrar a una base de datos), propone uno provisional y dilo como tal.

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
