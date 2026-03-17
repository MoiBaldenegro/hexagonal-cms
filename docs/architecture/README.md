# Arquitectura del sistema

## C4 Level 1 — Context

Muestra los actores externos y sistemas con los que interactúa el Testimonial CMS.

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Admin/Editor ──────────────────────► Testimonial CMS         │
│                                              │                  │
│   Visitante ─────────────────────────►       │                  │
│                                              ▼                  │
│                                     ┌────────────────┐         │
│                                     │  Sitio cliente │         │
│                                     │  (Embeds/API)  │         │
│                                     └────────────────┘         │
│                                              │                  │
│                             ┌────────────────┴──────────────┐  │
│                             ▼                               ▼  │
│                       YouTube API                      Cloudinary│
└─────────────────────────────────────────────────────────────────┘
```

**Actores:**
- **Admin / Editor** — gestiona, modera y publica testimonios desde el dashboard
- **Visitante** — consulta testimonios públicos vía web o embed

**Sistemas externos:**
- **Sitio web cliente** — consume testimonios vía REST API o snippet embed
- **YouTube API** — hosting y streaming de videos referenciados
- **Cloudinary** — almacenamiento, transformación y CDN de imágenes

---

## C4 Level 2 — Containers

Descomposición interna del sistema.

```
┌─── Testimonial CMS ──────────────────────────────────────────┐
│                                                              │
│  [Next.js SPA]          [Node.js + Express]                  │
│  Admin Dashboard  ────► REST API ──────────► PostgreSQL      │
│                              │                               │
│  [JS Widget]          [Auth Service]         Redis (cache)   │
│  Embed / iframe   ────► JWT + roles                          │
│                              │                               │
│  [Media Processor]           └──────────────► Search Engine  │
│  Validación + upload                         (PG FTS)        │
└──────────────────────────────┬───────────────────────────────┘
                               │
              ┌────────────────┴────────────────┐
              ▼                                 ▼
        YouTube API                        Cloudinary
```

| Contenedor | Tecnología | Responsabilidad |
|---|---|---|
| Admin Dashboard | Next.js 14 | UI de gestión, moderación y analítica |
| Embed Widget | JS snippet / iframe | Muestra testimonios en sitios externos |
| REST API | Node.js + Express | Lógica de negocio, endpoints públicos y privados |
| Auth Service | JWT + Redis | Autenticación y autorización por roles |
| Media Processor | Bull queue | Validación, resize y upload a Cloudinary |
| Base de datos | PostgreSQL 16 | Almacenamiento principal + Full Text Search |
| Cache | Redis | Queries frecuentes, sesiones, colas |

---

## NFRs clave

| Atributo | Objetivo |
|---|---|
| Carga del dashboard | < 3 s (LCP en 4G) |
| API pública (p95) | < 500 ms con cache |
| Tamaño máximo upload | 10 MB por archivo |
| Disponibilidad | 99.5% uptime mensual |
| Seguridad auth | JWT + httpOnly cookie + rate limiting |

---

> Los diagramas exportados de Miro/Excalidraw se guardan en [`diagrams/`](diagrams/).
