---
name: next-seguridad-web
description: Usar cuando hay que revisar la seguridad de una app de Next.js o React antes de lanzarla - XSS, dangerouslySetInnerHTML, CSP y cabeceras, secretos en el cliente, Server Actions sin autorización, proxy o middleware como única protección, límites de uso, dependencias vulnerables.
---

# Revisión de seguridad de una app Next.js

En una app de Next.js hay dos superficies, el navegador (lo que se descarga y se ejecuta) y el servidor (Server Actions, Route Handlers y Server Components). Una revisión útil demuestra cada hallazgo, arregla en el momento lo que no requiere decisión de producto y deja un informe en el repositorio.

1. Inventaría la superficie. Server Actions (`'use server'`), Route Handlers (`app/**/route.ts`), `proxy.ts` o `middleware.ts`, variables `NEXT_PUBLIC_`, usos de `dangerouslySetInnerHTML`, `eval` y redirecciones con URLs del usuario.
2. Revisa cada punto con esta tabla y anota archivo y línea.

   | Riesgo | Qué buscar |
   |---|---|
   | Autorización | cada acción y handler comprueba identidad y propiedad; el proxy o middleware no basta porque las acciones se pueden llamar directo |
   | Entrada | validación con Zod en el servidor, también en acciones que "solo usa el formulario" |
   | Abuso | límite de intentos y de uso en acciones públicas (registro, reservas, llamadas a IA); un límite en memoria no sirve con varias instancias |
   | Tokens y códigos | aleatorios de 128 bits o más, firmas HMAC completas comparadas con `timingSafeEqual` |
   | XSS | `dangerouslySetInnerHTML` solo con HTML saneado (DOMPurify) o nunca; URLs `javascript:` en `href` |
   | Secretos | nada secreto con `NEXT_PUBLIC_`; módulos con secretos llevan `import 'server-only'` |
   | Cabeceras | CSP con nonce, `X-Content-Type-Options`, `Referrer-Policy`, `frame-ancestors` |
   | Errores | sin trazas ni mensajes internos al cliente |
   | Dependencias | `npm audit --omit=dev` y versiones de `next` con parches de seguridad |

3. Las Server Actions no se llaman fácil con `curl`; pruébalas importando la función o su servicio en un test con datos ajenos. Demuestra cada hallazgo en local (`npm run build && npm start`). Llama la acción con datos ajenos, inyecta `<img src=x onerror=alert(1)>`, revisa las cabeceras con `curl -I`. Lo que no se pudo demostrar queda como posible.
4. Si las reglas del repositorio prohíben tocar un archivo (`next.config.ts`) o fijan un formato en tests públicos, ese arreglo queda pendiente y se explica. Arregla de una vez lo que no cambia el producto (comparación en tiempo constante, longitud del token, cabeceras que faltan, `server-only`) con su test. Lo que sí lo cambia (exigir cuenta para reservar, límites) se propone.
5. Escribe `SECURITY_REVIEW.md` con resumen de una línea, hallazgos con gravedad, evidencia y estado (arreglado o pendiente) y el orden recomendado.
6. Corre typecheck, lint, test y build después de los arreglos y muestra la salida.

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
