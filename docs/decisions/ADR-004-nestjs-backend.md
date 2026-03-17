# ADR-004 — NestJS como framework de backend

**Fecha:** 2025-03-17
**Estado:** `aceptado`  
**Autores:** equipo técnico

---

## Contexto

El backend necesita exponer una API REST documentada, gestionar roles y permisos, procesar uploads de media de forma asíncrona y mantener una arquitectura que escale con el equipo.

Se evaluaron:
- **Express** — minimalista, total libertad, pero sin convenciones claras para proyectos medianos/grandes
- **NestJS** — framework opinionado sobre Node.js con arquitectura modular, DI nativo e integración con TypeORM
- **Fastify** — más rápido que Express, pero menos ecosistema para las integraciones requeridas

## Decisión

Usar **NestJS** como framework principal del backend.

La arquitectura sigue los módulos de NestJS: `TestimonialsModule`, `AuthModule`, `MediaModule`, `CategoriesModule`, etc. Cada módulo encapsula su controlador, servicio y repositorio de TypeORM.

## Consecuencias

**Positivas:**
- Arquitectura modular y opinionada — reduce decisiones de estructura en el equipo
- Inyección de dependencias nativa facilita testing unitario con mocks
- Decoradores de Guards y Interceptors simplifican la validación de roles (`@Roles('admin')`, `@UseGuards(SupabaseAuthGuard)`)
- Integración oficial con TypeORM (`@nestjs/typeorm`), Bull (`@nestjs/bull`) y Swagger (`@nestjs/swagger`) — todo dentro del mismo ecosistema
- `@nestjs/swagger` genera el spec OpenAPI automáticamente desde los decoradores de los DTOs y controladores

**Negativas / trade-offs:**
- Más boilerplate inicial comparado con Express para endpoints simples
- Curva de aprendizaje si el equipo no está familiarizado con el patrón de módulos y DI de NestJS
- El CLI de NestJS genera muchos archivos — definir convenciones de estructura desde el inicio

**Riesgos:**
- NestJS abstrae mucho el ciclo de request/response; cuando algo falla, el debugging puede ser menos directo que en Express — conocer bien los Interceptors y Exception Filters antes de ir a producción
