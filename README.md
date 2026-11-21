# Crea tu frontend con React y Next.js asistido por IA

Curso en línea de tres unidades. En cada una construyes una app web con React y TypeScript, con tests desde el primer paso, y trabajas con un agente de código sin perder el control de lo que entra a tu proyecto: le pides el código, revisas el diff y lo compruebas con los tests.

Este repositorio es el índice del curso. El código de cada unidad está en su propia plantilla de GitHub, con el proyecto base, los tests y la integración continua listos.

[English version](README.en.md)

## Las unidades

| Unidad | Plantilla | Qué construyes |
| :--- | :--- | :--- |
| 1 | [`frontend-course-u1-gearforge`](https://github.com/ykro/frontend-course-u1-gearforge) | **GearForge Studio**, un configurador de workstations: React 19, TypeScript estricto, estado derivado, Tailwind v4, shadcn/ui, React Hook Form, Zod 4 y Testing Library. |
| 2 | [`frontend-course-u2-pulseboard`](https://github.com/ykro/frontend-course-u2-pulseboard) | **PulseBoard**, el tablero de tareas de un equipo: TanStack Query, cliente tipado desde OpenAPI, Firebase Auth, Firestore en vivo y MSW. |
| 3 | [`frontend-course-u3-eventpass`](https://github.com/ykro/frontend-course-u3-eventpass) | **EventPass**, un catálogo de charlas con inscripción: Next.js 16, Server Components, Server Actions, `useActionState`, `useOptimistic` y despliegue en Vercel. |

Cada unidad termina con un reto: un contrato escrito, sin código guiado, que verificas con los tests de la plantilla en tu repositorio.

## Cómo empezar

1. Abre la plantilla de la unidad y elige **Use this template > Create a new repository**.
2. Ponle tu propio nombre (no el de la plantilla) y elige visibilidad **Private**: así tu solución del reto no queda a la vista de cualquiera.
3. Clona tu repositorio y, en su raíz:

   ```bash
   npm ci     # instala las versiones exactas de package-lock.json
   npm test   # al inicio: la demo en verde; el taller y el reto en rojo
   ```

El `README.md` de cada plantilla explica cómo instalar y correr el proyecto. El contrato del reto, qué archivos puedes modificar y la lista para revisar tu trabajo están en la guía del reto de cada unidad, en el material del curso.

## Requisitos

- Node.js 24 (cada plantilla trae su `.nvmrc`), de preferencia con un gestor de versiones como `nvm` o `fnm`.
- Git y una cuenta gratuita de GitHub.
- Un editor con un agente de código y las extensiones de ESLint y Prettier, y Chrome o Edge.
- Unidad 3: una cuenta de Vercel (plan Hobby). Firebase es opcional en la Unidad 2.
- Haber programado antes en JavaScript o TypeScript y usar la terminal a nivel básico.

## Nivelación rápida antes de empezar (On-Ramp ~60 min)

Si sientes que el curso arranca rápido o nunca has trabajado con HTML semántico, React, peticiones asíncronas o Next.js, dedica una hora a este repositorio antes de abrir la Unidad 1:

- **Práctica paso a paso:** [`ykro/frontend-onramp`](https://github.com/ykro/frontend-onramp) (accesibilidad con Testing Library, Flexbox/Grid, estado inmutable en React, `fetch`/`AbortController` y frontera `"use client"`).
- **Solución explicada archivo por archivo:** [`ykro/frontend-onramp-solution`](https://github.com/ykro/frontend-onramp-solution).
- **Índice de los tres on-ramp (Backend, Frontend y Android):** [`ykro/courses-onramp`](https://github.com/ykro/courses-onramp).

## Paquete de 15 skills profesionales

Este repositorio incluye [`skills/`](skills/README.md), un paquete de **15 skills de producción para React y Next.js** pensado para tu trabajo profesional fuera del curso (TypeScript estricto, accesibilidad, estado derivado, formularios con Zod, TanStack Query, pruebas con Testing Library, Server/Client Components, Server Actions seguras, caché y revalidación, secretos, Web Vitals, seguridad web y despliegue en Vercel). Instálalas en tu carpeta personal de skills (`~/.claude/skills/`, `~/.gemini/antigravity/skills/` o `~/.agents/skills/`), nunca dentro del repo de una unidad.
