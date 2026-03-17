# ADR-003 — Supabase Auth para autenticación

**Fecha:** 2025-03-17
**Estado:** `aceptado`  
**Autores:** equipo técnico

---

## Contexto

El sistema tiene tres roles: `admin`, `editor` y `visitante`. El dashboard requiere sesiones persistentes. La API pública debe ser accesible sin autenticación para los embeds.

Se evaluaron:
- **JWT manual + refresh tokens** — control total, pero requiere implementar y mantener toda la lógica de auth
- **Supabase Auth** — auth gestionada integrada con la base de datos PostgreSQL que ya usamos en Supabase
- **Auth0 / Clerk** — solución gestionada robusta, pero añade dependencia externa y costo adicional al stack ya definido

## Decisión

Usar **Supabase Auth** para gestionar autenticación y sesiones.

Los roles (`admin`, `editor`, `visitante`) se implementan con **Row Level Security (RLS)** de PostgreSQL y custom claims en el JWT que emite Supabase. El backend NestJS valida los tokens JWT de Supabase en cada request usando el middleware de verificación con la clave pública de Supabase.

## Consecuencias

**Positivas:**
- Auth integrada con la misma instancia de PostgreSQL en Supabase — los roles y permisos viven junto a los datos
- Sin servidor de auth propio que mantener; Supabase gestiona rotación de tokens, sesiones y seguridad
- SDK oficial para React (`@supabase/supabase-js`) simplifica login, logout y manejo de sesión en el frontend
- RLS permite proteger datos directamente en la base de datos como capa adicional de seguridad
- Magic links y OAuth social disponibles sin configuración extra si se necesitan en el futuro

**Negativas / trade-offs:**
- Dependencia de Supabase como proveedor — un outage de Supabase afecta la autenticación
- Los custom claims de rol requieren un Supabase Edge Function o trigger para asignarse automáticamente al registrar usuarios
- Menos control sobre el flujo de auth comparado con JWT manual

**Riesgos:**
- Los JWT de Supabase expiran cada hora por defecto; asegurarse de que el cliente refresca el token correctamente con `onAuthStateChange`
- Validar que RLS esté correctamente configurada antes de ir a producción — una regla mal definida puede exponer datos de otros tenants
