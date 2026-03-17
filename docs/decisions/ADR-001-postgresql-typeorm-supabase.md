# ADR-001 — PostgreSQL con TypeORM en Supabase

**Fecha:** 2025-03-17
**Estado:** `aceptado`  
**Autores:** equipo técnico

---

## Contexto

El sistema necesita almacenar testimonios con relaciones claras: autores, categorías, tags, estado de moderación y métricas de engagement. También se requiere búsqueda por texto sobre el contenido de los testimonios.

Se evaluaron:
- **PostgreSQL + Prisma** — ORM con excelente DX y migraciones declarativas
- **PostgreSQL + TypeORM** — ORM maduro, integración nativa con NestJS vía decoradores
- **MongoDB** — documental, flexible en schema, pero rompe el modelo relacional ya definido

## Decisión

Usar **PostgreSQL 16 en Supabase** como base de datos principal con **TypeORM** como ORM, integrado con NestJS a través del módulo `@nestjs/typeorm`.

Las migraciones se gestionan con el CLI de TypeORM. Para búsqueda de texto se usa **PostgreSQL Full Text Search** en v1; si el volumen crece se evalúa Meilisearch.

## Consecuencias

**Positivas:**
- TypeORM se integra de forma nativa con NestJS mediante decoradores (`@Entity`, `@Column`, `@ManyToMany`, etc.) — sin fricción entre el ORM y el framework
- Supabase provee PostgreSQL gestionado con backups automáticos, dashboard visual de la BD y Row Level Security integrada
- FTS nativo de PostgreSQL cubre búsqueda sin añadir un servicio externo en v1
- Supabase también expone la BD vía PostgREST — útil para prototipar consultas rápidas sin tocar el backend

**Negativas / trade-offs:**
- TypeORM tiene una API más verbosa que Prisma para queries complejas
- Las migraciones de TypeORM requieren más cuidado que las de Prisma en entornos de producción — usar siempre `typeorm migration:generate` en lugar de `synchronize: true`
- `synchronize: true` **nunca debe usarse en producción** — destruye datos si hay cambios de schema

**Riesgos:**
- Supabase tiene límites de conexiones por plan; usar **PgBouncer** (incluido en Supabase) para connection pooling en producción
- Validar compatibilidad de versión entre TypeORM y la versión de PostgreSQL de Supabase antes de iniciar
