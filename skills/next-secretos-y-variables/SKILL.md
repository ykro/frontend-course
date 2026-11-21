---
name: next-secretos-y-variables
description: Usar cuando una app de Next.js necesita una API key o secreto (Gemini, OpenAI, Stripe, una base de datos), hay variables con NEXT_PUBLIC_, se pregunta "¿dónde pongo la API key?", o hay que configurar variables de entorno en local y en Vercel.
---

# Secretos y variables de entorno en Next.js

Toda variable que empieza con `NEXT_PUBLIC_` se copia al JavaScript del navegador y cualquiera la puede leer. Un secreto nunca lleva ese prefijo, se lee solo en código de servidor y ese código se marca con `import 'server-only'` para que el build falle si alguien lo importa desde el cliente.

1. Busca si el proyecto ya tiene un patrón (`lib/env.ts`, `getServerConfig`, `server-only`). Si existe, úsalo; no crees otro. Si ya está todo conectado, la respuesta es decir dónde poner el valor y cómo comprobarlo.
2. Clasifica cada variable. Pública (URL del sitio, id de analítica) con `NEXT_PUBLIC_`. Secreta (llaves de API, conexión a base de datos, firma de sesiones) sin prefijo.
3. Una variable obligatoria detiene la app si falta; una opcional (por ejemplo, la función con IA) la apaga con un mensaje. Valida las variables del servidor al arrancar con Zod en un módulo `server-only`, para que falte una y el error diga cuál.
4. Llama al servicio externo desde el servidor (Server Action, Route Handler o Server Component), nunca desde un componente cliente. El cliente recibe solo el resultado.
5. Pon los valores en `.env.local` (ya ignorado por git) y los nombres sin valores en `.env.example`. En Vercel, Settings, Environment Variables, por entorno (Production, Preview, Development), y Redeploy después de cambiarlas.
6. Restringe la llave en el proveedor (solo el API que usa, cuota diaria, alerta de presupuesto) y define qué pasa si falla o se agota (mensaje y respaldo sin IA).
7. Si respondes la pregunta "dónde va", respóndela primero en tres líneas. Si además implementas, conecta todo (acción, UI y prueba) o no dejes código a medias.
8. Comprueba. `npm run build` y luego busca la llave en el resultado (`grep -r "AIza" .next/static` debe salir vacío; con valores falsos de prueba en el build, busca ese valor; si el proyecto trae un script de sondeo, úsalo), y corre los tests con la llamada externa simulada.

```ts
// lib/server-env.ts
import 'server-only'
import { z } from 'zod'

const schema = z.object({ GEMINI_API_KEY: z.string().min(20) })

export function serverEnv() {
  const parsed = schema.safeParse(process.env)
  if (!parsed.success) throw new Error(`Faltan variables: ${parsed.error.issues.map(i => i.path.join('.')).join(', ')}`)
  return parsed.data
}
```

Si una llave ya estuvo en el cliente o en git, se revoca en el proveedor y se crea otra; borrarla del código no basta.

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
