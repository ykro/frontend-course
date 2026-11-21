---
name: react-datos-con-tanstack-query
description: Usar cuando una app React lee o cambia datos de un servidor con TanStack Query (React Query) - useQuery, useMutation, llaves de consulta, invalidación, actualización optimista, datos que no se refrescan, peticiones duplicadas, o useEffect con fetch que conviene reemplazar.
---

# Datos de servidor con TanStack Query

Los datos del servidor no son estado local; tienen dueño, caducan y pueden fallar. Se leen con `useQuery` bajo llaves jerárquicas, se cambian con `useMutation` y después se invalida por prefijo. La actualización optimista guarda una foto para regresar si el servidor responde con error.

1. Antes de escribir, confirma que el servidor tiene la operación. Revisa el contrato (OpenAPI, tipos generados o la documentación del API). Si la ruta no existe, detente y pregunta; no escribas código contra una ruta inventada ni edites a mano archivos generados.
2. Si el proyecto ya tiene fábrica de llaves y reglas de invalidación, úsalas tal cual; el ejemplo de abajo es la forma general. Si no, centraliza las llaves en una fábrica (`taskKeys.all`, `taskKeys.list(boardId)`, `taskKeys.detail(id)`), para que invalidar `taskKeys.all` alcance todas las variantes.
3. Cada consulta vive en un hook (`useTasks(boardId)`) que recibe todo lo que cambia la respuesta como parte de la llave. Define `staleTime` según qué tan rápido cambian los datos, no en cero por costumbre.
4. Para cambios, un hook con `useMutation`. Si el usuario debe verlo al instante, usa la forma optimista de abajo. Si no, basta `onSuccess` con `invalidateQueries`.
5. En la UI maneja los estados `isPending`, `isError` (con un mensaje y reintento) y vacío. Mientras la mutación corre, el control queda deshabilitado o muestra el estado.
6. Si la lista tiene variantes (filtros, páginas), actualiza todas con `getQueriesData` y `setQueriesData` sobre el prefijo de listas, e invalida el prefijo más específico que cubra el cambio. Prueba sin red. Con MSW (o el doble del proyecto) responde éxito y error, y comprueba que con error la tarea vuelve a como estaba y aparece el mensaje.
7. Corre `npm run typecheck`, `npm test` y `npm run lint` (con `@tanstack/eslint-plugin-query`). Si el typecheck falla porque falta algo del contrato, el cambio no está listo.

```ts
export function useToggleTask(boardId: string) {
  const qc = useQueryClient()
  return useMutation({
    mutationFn: (v: { id: string; done: boolean }) => api.updateTask(boardId, v.id, { done: v.done }),
    onMutate: async (v) => {
      await qc.cancelQueries({ queryKey: taskKeys.list(boardId) })
      const previous = qc.getQueryData<Task[]>(taskKeys.list(boardId))
      qc.setQueryData<Task[]>(taskKeys.list(boardId), old =>
        old?.map(t => (t.id === v.id ? { ...t, done: v.done } : t)))
      return { previous }
    },
    onError: (_err, _v, ctx) => {
      if (ctx?.previous) qc.setQueryData(taskKeys.list(boardId), ctx.previous)
    },
    onSettled: () => qc.invalidateQueries({ queryKey: taskKeys.all }),
  })
}
```

## Errores comunes

| Error | Corrección |
|---|---|
| `useEffect` con `fetch` y `setState` | `useQuery` |
| Llaves armadas a mano en cada archivo | fábrica de llaves |
| Optimista sin `cancelQueries` | una respuesta vieja pisa el cambio |
| Copiar `data` a `useState` | usar `data` directo o derivar |

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
