---
name: react-estado-derivado
description: Usar cuando un componente de React no se actualiza al hacer clic, muestra totales o listas desfasadas, tiene useEffect que copia props o calcula valores en estado, muta arrays u objetos (push, item.qty++), usa key={index}, o hay renders de más o "stale closures".
---

# Estado mínimo y lo demás derivado

React vuelve a renderizar cuando recibe un valor nuevo (`Object.is` distinto). Si se muta el array y se pasa el mismo, no pasa nada. Y todo lo que se puede calcular a partir de props o estado se calcula al renderizar, no se guarda en otro estado con un `useEffect` que llega un render tarde.

1. Lista cada `useState` y cada `useEffect` del componente. Para cada estado pregunta si se puede calcular con otros valores; si sí, se borra y se calcula en el render (`const total = items.reduce(...)`). Si es caro, `useMemo`.
2. Busca efectos que sincronizan estado con estado o con props (`useEffect(() => setX(f(y)), [y])`). Se reemplazan por cálculo en el render. Los efectos quedan solo para sincronizar con algo externo (suscripciones, DOM, red sin librería).
3. Decide quién es el dueño de cada dato. Si el padre manda `items`, el hijo no lo copia a su estado; usa la prop y avisa cambios con un callback. Si de verdad necesita un borrador local, que el padre lo reinicie con `key`. Si no está claro quién es el dueño, propón uno, pregúntalo y mientras tanto conserva las props públicas del componente; el ejemplo de abajo cambia la API solo para ilustrar.
4. Reemplaza toda mutación por copias. `setList(prev => [...prev, item])`, `prev.map(i => i.id === id ? { ...i, qty: i.qty + 1 } : i)`, nunca `push`, `splice`, `sort` en sitio ni `item.qty++` sobre un objeto que vino de props. Usa la forma con función (`prev =>`) cuando el nuevo valor depende del anterior.
5. Usa una `key` estable del dato (`item.id`). `key={index}` mezcla estado entre filas al filtrar o reordenar.
6. Lo que se puede tocar es un `<button>` (o un enlace si navega), no un `div` con `onClick`, para que funcione con teclado y lector de pantalla.
7. No cambies comportamiento que no se pidió (por ejemplo, el filtro a insensible a mayúsculas). Si te parece mejor, propónlo.
8. Escribe una prueba que reproduzca el bug (clic, la lista y el total cambian) y verifica que falla con el código viejo y pasa con el nuevo. Corre `npm run typecheck`, `npm test` y `npm run lint` (con `eslint-plugin-react-hooks`). Si el archivo está suelto, sin proyecto, pruébalo dentro de un proyecto mínimo con Vite y Vitest y dilo.

```tsx
function Wishlist({ items, onChange }: { items: readonly Item[]; onChange: (next: Item[]) => void }) {
  const [query, setQuery] = useState('')
  const filtered = items.filter(i => i.name.includes(query))
  const total = items.reduce((s, i) => s + i.price * i.qty, 0)

  function addOne(id: string) {
    onChange(items.map(i => (i.id === id ? { ...i, qty: i.qty + 1 } : i)))
  }

  return (
    <>
      <label>Buscar <input value={query} onChange={e => setQuery(e.target.value)} /></label>
      <ul>
        {filtered.map(i => (
          <li key={i.id}><button type="button" onClick={() => addOne(i.id)}>{i.name} x{i.qty}</button></li>
        ))}
      </ul>
      <p>Total: {total}</p>
    </>
  )
}
```

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
