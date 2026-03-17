# ADR-002 — React + Vite + React Router DOM para frontend

**Fecha:** 2025-03-17
**Estado:** `aceptado`
**Autores:** equipo técnico

---

## Contexto

El sistema tiene dos superficies de frontend:

1. **Admin Dashboard** — interfaz privada de gestión, moderación y analítica. Requiere UX fluida y formularios complejos.
2. **Widget público / Embed** — componente que se inyecta en sitios de clientes.

Se evaluaron:
- **Next.js 14** — SSR + Client Components, un solo framework para ambas superficies
- **React + Vite + React Router DOM** — SPA pura, build rápido, sin opiniones de framework
- **Astro** — excelente para contenido estático pero menos adecuado para el dashboard interactivo

## Decisión

Usar **React 18 + Vite** como bundler y **React Router DOM v6** para el routing del dashboard.

El widget embed se distribuye como un snippet JS independiente que hace fetch a la API pública y renderiza los testimonios en el sitio del cliente.

## Consecuencias

**Positivas:**
- Build extremadamente rápido con Vite (HMR instantáneo en desarrollo)
- Sin opiniones de framework — estructura de carpetas libre
- React Router DOM v6 cubre todas las necesidades de routing del dashboard (layouts anidados, loaders, rutas protegidas)
- Ecosistema familiar para la mayoría de devs frontend

**Negativas / trade-offs:**
- SPA pura: los testimonios del widget no son indexables por buscadores desde el dashboard. Si el SEO del embed es crítico, considerar SSR en una iteración futura
- Sin SSR/ISR nativo; el tiempo de carga inicial depende del bundle size — aplicar code splitting por ruta

**Riesgos:**
- El widget embed como snippet JS separado implica mantener dos builds (dashboard + widget); documentar el proceso de release de ambos
