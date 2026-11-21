---
name: react-pruebas-testing-library
description: Usar cuando hay que escribir o arreglar pruebas de componentes React con Vitest o Jest y Testing Library - qué probar, consultas por rol, user-event, mocks de red con MSW, tests frágiles que se rompen al refactorizar, o tests que pasan aunque la app esté rota.
---

# Pruebas de componentes con Testing Library

Se prueba lo que ve y hace la persona, no cómo está hecho el componente. Se busca por rol y nombre accesible, se interactúa con `user-event` y se afirma lo que aparece en pantalla. Una prueba vale si falla cuando el comportamiento se rompe.

1. Prueba el componente que existe. Si el pedido dice "asume que ya se corrigió" y la versión corregida no está en el repositorio, pregunta cuál usar; no reescribas el componente para probar tu versión. Si la persona insiste, escribe las pruebas contra el comportamiento esperado, entrega también la corrección mínima que las hace pasar y dilo.
2. Lee la configuración (`vitest.config`, `setupTests`, si usa `jsdom` o `happy-dom`, si ya hay `@testing-library/jest-dom` y MSW). Integra lo que falte en la configuración real, no en archivos de ejemplo aparte.
3. Lista los comportamientos que importan al usuario. Qué se muestra al cargar, qué pasa al escribir, al hacer clic, con lista vacía, con error de red y con datos de borde.
4. Busca con `getByRole('button', { name: /agregar/i })`, `getByLabelText` o `getByText`. `getByTestId` solo si no hay otra forma. Si no puedes encontrar un control por rol, eso es un problema de accesibilidad que conviene reportar.
5. Interactúa con `const user = userEvent.setup()` y `await user.click(...)`, `await user.type(...)`. Usa `findBy` para lo que aparece después.
6. Afirma resultados visibles (`toHaveTextContent`, `toBeInTheDocument`, `toHaveBeenCalledWith` en callbacks que son parte del contrato). Para comprobar inmutabilidad, verifica que el arreglo recibido no cambió (`Object.freeze` en los datos de prueba hace que una mutación truene; para que compile, la prop debe aceptar `readonly Item[]`).
7. Para red, MSW con handlers por test (éxito, error 500, lento). Nada de mocks del `fetch` global repartidos en cada archivo.
8. No fijes comportamiento que nadie decidió (si el filtro distingue mayúsculas). Pregunta o deja ese caso fuera y dilo.
9. Corre `npm test` y muestra la salida. Rompe a propósito una línea del componente para ver que al menos una prueba falla, y deshaz el cambio.

```tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { expect, test, vi } from 'vitest'

test('agregar uno avisa la cantidad nueva sin mutar la lista', async () => {
  const items = Object.freeze([Object.freeze({ id: 'a', name: 'Taza', price: 50, qty: 1 })])
  const onChange = vi.fn()
  const user = userEvent.setup()
  render(<Wishlist items={items} onChange={onChange} />)

  await user.click(screen.getByRole('button', { name: /taza x1/i }))

  expect(onChange).toHaveBeenCalledWith([{ id: 'a', name: 'Taza', price: 50, qty: 2 }])
  expect(screen.getByText('Total: 50')).toBeInTheDocument()
})
```

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
