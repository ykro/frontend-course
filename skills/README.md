# Skills de frontend con React y Next.js

Estas quince skills son para tu trabajo después del curso, en cualquier app con React, Next.js (App Router), TypeScript y Vercel. Cada una es una carpeta con un archivo `SKILL.md`. Tu asistente de código lee su descripción y la usa sola cuando tu pedido coincide; en Claude Code también la puedes llamar por su nombre, como `/react-revisar-codigo-generado`.

| Skill | Cuándo te ayuda |
|---|---|
| `react-estado-derivado` | componentes que no se actualizan, totales desfasados, mutaciones y `useEffect` de más |
| `react-typescript-estricto` | quitar `any`, `as` y `!`, y modelar estados con uniones discriminadas |
| `react-formularios-zod` | formularios con React Hook Form y Zod, reglas entre campos y errores accesibles |
| `react-datos-con-tanstack-query` | leer y cambiar datos con TanStack Query, llaves, invalidación y actualización optimista |
| `react-errores-y-carga` | estados de carga, error, vacío y no encontrado; `error.tsx` y `loading.tsx` |
| `react-pruebas-testing-library` | pruebas de componentes con Testing Library, consultas por rol y MSW |
| `react-accesibilidad` | modales, menús, foco, teclado y lectores de pantalla |
| `react-revisar-codigo-generado` | revisar un PR o código generado por IA antes de aceptarlo |
| `next-componentes-servidor-y-cliente` | decidir qué va en el servidor y qué en el cliente, y bajar el JavaScript |
| `next-server-actions-seguras` | Server Actions que validan, autorizan y revalidan bien |
| `next-cache-y-revalidacion` | datos viejos en producción, `'use cache'`, etiquetas y revalidación |
| `next-secretos-y-variables` | dónde van las API keys y qué hace `NEXT_PUBLIC_` |
| `next-seguridad-web` | revisar XSS, cabeceras, CSP y autorización antes de lanzar |
| `next-rendimiento-web-vitals` | LCP, CLS e INP malos, imágenes, fuentes y bundle |
| `next-desplegar-en-vercel` | desplegar en Vercel y resolver builds que fallan allá y no en tu máquina |

Instálalas en tu carpeta personal y nunca dentro del repositorio de una unidad del curso. Un archivo agregado fuera de tus carpetas cuenta como sospecha de integridad en la rúbrica, y una carpeta `.claude/skills/` o `.agents/skills/` dentro del repo cuenta como eso. Descomprime `frontend-react-next-skills.zip` y copia las quince carpetas de `skills/` (cada una con su `SKILL.md`) al lugar de tu herramienta.

1. Claude Code usa `~/.claude/skills/`.
2. Antigravity usa `~/.gemini/antigravity/skills/`.
3. Gemini CLI usa `~/.gemini/skills/` y también reconoce `~/.agents/skills/`.
4. Codex reconoce `~/.agents/skills/`.
5. Cierra y vuelve a abrir la herramienta (o empieza una conversación nueva) y pregúntale "¿qué skills tienes?" para comprobarlo.

En Mac o Linux, dentro de la carpeta `skills` que sale al descomprimir el ZIP, el comando para Claude Code es `mkdir -p ~/.claude/skills && cp -R react-* next-* ~/.claude/skills/`. En Windows, copia las carpetas con el Explorador a `C:\Users\TU_USUARIO\.claude\skills\`.

En tus proyectos fuera del curso puedes ponerlas dentro del repo si tu equipo las quiere compartir. Si una skill dice algo distinto del `AGENTS.md` del proyecto, manda ese archivo, y lo que entregas sigue siendo tu responsabilidad.
