---
name: react-revisar-codigo-generado
description: Usar cuando hay que revisar un pull request, un diff o código generado por IA en un proyecto React o Next.js antes de aceptarlo ("¿lo acepto?", "revisa lo que hizo el agente", code review de componentes, hooks, Server Actions o configuración).
---

# Revisar código React o Next generado

El código generado compila y se ve bien en la demo. Se revisa contra lo que pidió la tarea y las reglas del proyecto, buscando los defectos que la IA repite, con evidencia de typecheck, lint, tests y build sobre el cambio.

1. Lee el pedido, `AGENTS.md` si existe y el diff (`git diff main...RAMA`). Si solo hay un diff suelto, pregunta a qué proyecto pertenece, aplícalo en una copia (si no trae contexto, reconstrúyelo a mano y dilo) o revísalo en lectura y dilo en la evidencia. Si no sabes si la llave ya corrió en un preview o en un repositorio, trátala como expuesta.
2. Revisa primero lo que no tiene vuelta atrás. Una llave secreta en el cliente (`NEXT_PUBLIC_` o `VITE_` con un secreto, una llave escrita en el código) es bloqueante, y si el cambio llegó a correr en un preview o a un repositorio, la llave se revoca y se cambia, no solo se borra.
3. Recorre la tabla y anota cada hallazgo con archivo y línea, qué le pasa al usuario y cómo se corrige.

   | Área | Qué buscar |
   |---|---|
   | Seguridad | `dangerouslySetInnerHTML` con datos de usuarios, Server Actions sin validación ni autorización, ids o `userId` tomados del formulario, secretos fuera de `server-only` |
   | Estado | `useEffect` que copia props o calcula, mutaciones (`push`, `item.x =`), `key={index}` |
   | Datos | `fetch` en efectos en lugar de la librería del proyecto, revalidación de más (`revalidatePath('/')`) o de menos |
   | Servidor y cliente | `'use client'` en una página entera, librerías pesadas en el cliente |
   | Tipos | `any`, `as` que esconden errores, `!` sin razón |
   | Accesibilidad | `div` con `onClick`, imágenes sin `alt`, inputs sin `label` |
   | Pruebas | ¿hay tests que fallarían sin el cambio? ¿se debilitó alguno? |

4. Corre `npm run typecheck`, `npm run lint`, `npm test` y `npm run build` en la rama y compara con `main`.
5. Entrega el veredicto en una línea y después la lista. Escribe aparte el mensaje para el agente o el comentario del PR, listo para pegar, con los bloqueantes primero.

```
Veredicto: no aceptar | aceptar con cambios | aceptar
Bloqueantes
1. archivo:linea - que le pasa al usuario - correccion
Importantes
Menores
Evidencia: typecheck, lint, test y build (resultado de cada uno)
Acciones fuera del codigo: revocar llave X, etc.
```

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
