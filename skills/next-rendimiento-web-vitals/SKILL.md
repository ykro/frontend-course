---
name: next-rendimiento-web-vitals
description: Usar cuando una app de Next.js o React es lenta - puntaje bajo de Lighthouse o PageSpeed Insights, LCP, CLS o INP malos, TTFB alto, imágenes o fuentes pesadas, bundle grande, o "¿por dónde empiezo a optimizar?".
---

# Rendimiento y Core Web Vitals en Next.js

Se optimiza la métrica que falla, en la página que importa, con datos de usuarios reales cuando existen. Primero se mide, se encuentra el elemento o la tarea culpable y se cambia una cosa a la vez. Lighthouse en móvil simula un teléfono lento con red lenta; por eso da peor que en tu computadora.

1. Si no tienes la URL ni el código, pídelos primero y da un solo paso concreto mientras tanto. Con la URL, córrela en PageSpeed Insights; sin URL pública, corre Lighthouse en modo móvil sobre `npm run build && npm start`. Anota los datos de campo (CrUX, si hay) y los de laboratorio por separado. Las metas son LCP menor a 2.5 s, INP menor a 200 ms y CLS menor a 0.1.
2. Encuentra al culpable de cada métrica. En el reporte, "Largest Contentful Paint element" dice qué elemento es el LCP y "Avoid large layout shifts" qué se movió. En Chrome DevTools, Performance, con CPU 4x más lenta, se ven las tareas largas que dañan INP.
3. El primer cambio es el del elemento LCP.

   | Causa del LCP | Arreglo |
   |---|---|
   | Imagen grande o sin prioridad | `next/image` con `preload` (`priority` antes de Next 16), `sizes` correcto y formato moderno |
   | Video o carrusel arriba | imagen fija como póster; el video carga después |
   | Texto con fuente externa | `next/font` (se sirve desde tu dominio, sin salto) |
   | TTFB alto (más de 800 ms) | datos lentos en el servidor, región lejana de la base de datos, página dinámica que podría cachearse |
   | Render en el cliente | la página es `'use client'` y espera JavaScript; pásala a Server Component |

4. CLS. Toda imagen, video, iframe y anuncio con tamaño reservado (`width` y `height` o `aspect-ratio`), fuentes con `next/font`, y nada que se inserte arriba del contenido después de cargar.
5. INP. Divide el trabajo pesado de los manejadores de eventos, usa `startTransition` para actualizaciones no urgentes y saca JavaScript de terceros (chat, analítica, píxeles) con `next/script` y `strategy="lazyOnload"`.
6. Revisa el bundle y quita librerías grandes del cliente. Con Turbopack (Next 16) el build ya no imprime tamaños; usa `next analyze` (antes `next experimental-analyze`). Con webpack, la tabla de `npm run build` o `@next/bundle-analyzer`.
7. Después de cada cambio, vuelve a medir con la misma herramienta y el mismo dispositivo, tres corridas, y reporta la mediana antes y después.
8. Deja una forma de vigilarlo. Vercel Speed Insights o `useReportWebVitals` enviando a tu analítica, para ver datos reales y no solo laboratorio.

