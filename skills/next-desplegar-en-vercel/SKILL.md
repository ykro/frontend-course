---
name: next-desplegar-en-vercel
description: Usar cuando hay que desplegar una app de Next.js en Vercel o el despliegue falla - build que pasa en local y falla en Vercel, variables de entorno, datos que se pierden o difieren entre peticiones, previews, dominios, límites de funciones, o "¿qué reviso antes de desplegar?".
---

# Desplegar Next.js en Vercel

Vercel construye desde cero con el lockfile y la versión de Node que le digas, y corre el servidor como funciones sin memoria compartida que se apagan solas. Lo que depende de tu máquina (un archivo sin commit, mayúsculas en un import en macOS, una variable de tu `.env.local`) o de la memoria del proceso falla allá.

1. Reproduce el build de Vercel en local, en limpio. `rm -rf .next node_modules && npm ci && npm run build`. Si falla aquí, ya tienes la causa.
2. Para reproducir el entorno de Vercel, `vercel pull` (baja variables y configuración) y `vercel build`. Revisa mayúsculas contra Git con `git ls-files` comparando cada ruta importada. Si el build falla solo en Vercel, pide el log (Deployments, el despliegue, Build Logs) o léelo con `vercel inspect URL --logs`, y busca la primera línea de error, no la última.

   | Error en el log | Causa común |
   |---|---|
   | `Module not found` con un archivo que sí existe | mayúsculas distintas en el import (macOS no distingue, Linux sí) o archivo sin commit |
   | Errores de tipos que en local no salen | versiones distintas por no usar el lockfile, o `ignoreBuildErrors` solo en local (desde Next 16 el build ya no corre lint; córrelo aparte) |
   | Falta una variable | no está en Vercel para ese entorno (Production o Preview) |
   | Versión de Node | `engines.node` en `package.json` distinto del configurado en Vercel |

3. Revisa las variables. Una que falta puede no romper el build y sí la primera acción que la usa. Todas las de `.env.example` existen en Vercel en cada entorno que las necesita, las secretas sin `NEXT_PUBLIC_`, y hubo Redeploy después de cambiarlas.
4. Busca estado en memoria. Arreglos, `Map` o contadores a nivel de módulo usados como base de datos, sesiones o límites de uso. En Vercel cada instancia tiene los suyos y se borran al apagarse; por eso "las reservas se pierden". Se reemplazan por una base de datos o un almacén (Postgres, Redis, Firestore) desde el Marketplace de Vercel. Mientras tanto, dilo claramente como demo y no como producto.
5. Revisa lo que no corre en funciones. Escribir archivos en disco (solo `/tmp`, efímero), tareas largas (límite de duración de la función), WebSockets propios.
6. Despliega primero a Preview, prueba el flujo principal en esa URL (crear, recargar, abrir en otro navegador) y solo después promueve a Production.
7. Después de Production, revisa los logs de funciones (Logs en el proyecto) mientras haces el flujo una vez y configura el dominio.
8. Reporta qué causó la falla con la línea del log, qué cambiaste y la prueba en Preview.

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
