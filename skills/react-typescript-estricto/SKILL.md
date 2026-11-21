---
name: react-typescript-estricto
description: Usar cuando un proyecto TypeScript o React tiene tipos con muchos campos opcionales que dependen de un estado, any, as o ! para callar errores, ifs y ? por todas partes, datos externos sin validar, o hay que activar strict o modelar estados con uniones discriminadas.
---

# TypeScript estricto con uniones discriminadas

Si un campo solo existe en cierto estado, el tipo lo debe decir. Una unión discriminada por `estado` hace imposibles las combinaciones inválidas (un pedido enviado sin guía) y obliga a cubrir todos los casos con un `switch` exhaustivo. Los datos que vienen de fuera se validan en la frontera con Zod y de ahí sale el tipo.

1. Confirma `"strict": true` en `tsconfig.json` (y conviene `noUncheckedIndexedAccess`). Corre `npx tsc --noEmit` y anota cuántos errores hay antes.
2. Lista los estados reales con la persona que conoce el negocio y qué datos existen en cada uno. Lo que no sepas (¿hay reembolso?, ¿se cancela un pedido enviado?) se pregunta, y mientras tanto entregas la versión con los estados que sí conoces.
3. Escribe la unión con un discriminante literal. Cada variante tiene solo sus campos y como obligatorios.
4. No cambies de paso la representación de otros campos (unidades del dinero, ids) sin decirlo; si `total: any` debe ser número, `number` en la unidad que ya usa el código, y lo demás se propone.
5. Cubre todos los casos con `switch` y un `default` que asigna a `never`. Al agregar un estado, el compilador marca cada lugar que falta.
6. Crea los valores con funciones constructoras por estado (`marcarPagado(pedido, fecha)`), que solo aceptan la transición válida.
7. Valida la frontera. Lo que llega del API, de `localStorage` o de un formulario pasa por un esquema de Zod con `z.discriminatedUnion` y el tipo sale de `z.infer`, para no mantener dos definiciones.
8. Migra por partes. Cambia el tipo, corre `tsc`, corrige archivo por archivo reemplazando `?.` y `!` por el `switch` o un estrechamiento (`if (p.estado === 'enviado')`). No uses `as` para callar errores.
9. Verifica que compila con `npx tsc --noEmit` y que los tests pasan, y reporta los errores antes y después.

```ts
type Pedido =
  | { estado: 'pendiente'; id: string; total: number }
  | { estado: 'pagado'; id: string; total: number; pagadoEn: Date }
  | { estado: 'enviado'; id: string; total: number; pagadoEn: Date; guia: string }
  | { estado: 'cancelado'; id: string; total: number; motivoCancelacion: string }

function etiqueta(p: Pedido): string {
  switch (p.estado) {
    case 'pendiente': return 'Esperando pago'
    case 'pagado': return `Pagado el ${p.pagadoEn.toLocaleDateString('es-GT')}`
    case 'enviado': return `En camino, guía ${p.guia}`
    case 'cancelado': return `Cancelado: ${p.motivoCancelacion}`
    default: {
      const nunca: never = p
      return nunca
    }
  }
}
```

Un constructor por transición se escribe así: `function marcarPagado(p: Extract<Pedido, { estado: 'pendiente' }>, pagadoEn: Date): Extract<Pedido, { estado: 'pagado' }> { return { ...p, estado: 'pagado', pagadoEn } }`.

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
