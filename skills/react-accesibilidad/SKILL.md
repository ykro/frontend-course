---
name: react-accesibilidad
description: Usar cuando hay que revisar o corregir la accesibilidad de componentes React o páginas web - modales, menús, formularios, botones hechos con div, imágenes sin alt, contraste, foco, navegación con teclado, lectores de pantalla, WCAG, o un reporte de axe o Lighthouse.
---

# Accesibilidad de componentes React

Primero HTML nativo (`button`, `a`, `label`, `dialog`), después ARIA solo para lo que el HTML no dice. Se comprueba con teclado, con un lector de pantalla o al menos con axe, y con una prueba automática que impide que vuelva el problema.

1. Pasa axe sobre el componente o la página (la extensión axe DevTools, o `axe-core` en un test con `const r = await axe.run(container)` y `expect(r.violations).toEqual([])`) y anota los hallazgos. En jsdom axe no calcula contraste; ese se revisa en el navegador. axe encuentra cerca de un tercio de los problemas; el resto se ve con teclado.
2. Recorre con teclado. Tab llega a todo lo que se puede tocar en un orden lógico, Enter y Espacio activan, Escape cierra lo que se abrió, y el foco siempre se ve (`:focus-visible`).
3. Corrige con esta tabla.

   | Problema | Corrección |
   |---|---|
   | `div` o `span` con `onClick` | `<button type="button">` o `<a href>` si navega |
   | Botón solo con icono | `aria-label` con la acción ("Cerrar") y el SVG con `aria-hidden` |
   | Input sin nombre | `<label htmlFor>` visible; el placeholder no es etiqueta |
   | Imagen | `alt` que dice lo que comunica, o `alt=""` si es decorativa |
   | Contraste | texto normal al menos 4.5 a 1 y grande 3 a 1; calcúlalo con los colores reales |
   | Objetivo táctil pequeño | al menos 24 por 24 px (WCAG 2.2) y 44 por 44 px como meta en móvil |
   | Título visual sin jerarquía | `h2` o el nivel que toque, no un `span` en negrita |
   | Errores de formulario | `aria-invalid` y `aria-describedby` hacia el mensaje |

4. Para modales usa `<dialog>` con `showModal()`. Da el foco atrapado, Escape, el fondo inerte y el rol correctos. `showModal()` enfoca el primer control; si quieres otro, ponle `autofocus`. Al cerrar, devuelve el foco al botón que lo abrió, y dale nombre con `aria-labelledby`. Si no puedes usar `dialog`, necesitas `role="dialog"`, `aria-modal`, `inert` en el resto de la página y la trampa de foco a mano.
5. Respeta `prefers-reduced-motion` en animaciones y no transmitas información solo con color.
6. jsdom no implementa `showModal` ni el cierre con Escape del `dialog`; usa un polyfill en el setup de pruebas o prueba en un navegador real (Playwright). Escribe una prueba con Testing Library que abra el componente, lo use solo con teclado (`user.keyboard('{Escape}')`) y pase axe sin violaciones.
7. Reporta lo corregido, cómo se probó (teclado, axe, lector de pantalla) y lo que no se pudo probar.

Si `AGENTS.md` u otras reglas del repositorio dicen algo distinto de esta skill, mandan las del repositorio; dilo en tu respuesta. Si el proyecto tiene un script que agrupa las comprobaciones (`npm run verify` o `npm run check`), úsalo en lugar de correr cada comando por separado. Si falla por partes que no te tocaban, corre los comandos sueltos y di qué queda en rojo y por qué.
