# docs/04-decisions-ADR.md — Decisiones técnicas (ADR) — MVP

> ADR = _Architecture Decision Record_.
> Este documento registra decisiones importantes y por qué se tomaron, para mantener coherencia y poder explicarlo (por ejemplo, en entrevistas).

---

## ADR-001 — Stack backend: NestJS + TypeScript

**Decisión:** Usar **NestJS** como framework principal y **TypeScript** como lenguaje.

**Motivo:**

- NestJS ofrece una arquitectura modular (modules/controllers/services) adecuada para proyectos “empresa real”.
- TypeScript mejora mantenibilidad, reduce errores y facilita refactors.
- Es un stack frecuente en ofertas laborales (Node + TS) y sirve para portafolio.

**Alternativas consideradas:**

- Express (más simple, pero más fácil de desordenar y requiere armar arquitectura manual).
- Node puro (educativo, pero no realista para un MVP profesional).

**Consecuencias:**

- Curva inicial un poco mayor que Express.
- Mejor orden y escalabilidad desde el inicio.

---

## ADR-002 — Base de datos: PostgreSQL

**Decisión:** Usar **PostgreSQL** como base de datos principal.

**Motivo:**

- Relacional, consistente y estándar en sistemas de negocio.
- Buen soporte para integridad referencial (FK, constraints, índices).
- Encaja con el modelo de casos/acciones y multi-tenant.

**Alternativas consideradas:**

- SQLite (muy simple, pero poco realista para producción).
- MongoDB (flexible, pero aquí las relaciones y consistencia son centrales).

**Consecuencias:**

- Requiere configuración inicial (DB + migrations/ORM).
- Modelo más sólido y profesional.

---

## ADR-003 — Entorno de desarrollo: Docker Compose

**Decisión:** Usar **Docker Compose** para levantar API + DB en desarrollo local.

**Motivo:**

- Reproducibilidad (misma configuración para cualquiera que clone el repo).
- Aislamiento y facilidad para mover el proyecto a otro servidor.
- Permite persistencia con volúmenes (PostgreSQL).

**Alternativas consideradas:**

- Instalar Postgres local fuera de Docker (más fricción, menos portable).
- Servicios administrados desde el inicio (innecesario para MVP).

**Consecuencias:**

- Se deben documentar puertos, volúmenes y variables de entorno.
- Facilita migración a servidor Linux más adelante.

---

## ADR-004 — Arquitectura: API REST (backend separado)

**Decisión:** Exponer funcionalidad como **API REST**.

**Motivo:**

- Separación clara entre backend y frontend.
- Facilita usar Postman/Swagger al inicio y agregar UI después.
- Es el patrón más común en aplicaciones empresariales.

**Alternativas consideradas:**

- Monolito fullstack con SSR (menos flexible para cambiar frontend).
- GraphQL (más complejo sin necesidad en MVP).

**Consecuencias:**

- Hay que definir contrato de endpoints y validaciones.
- Se requiere auth y manejo consistente de errores.

---

## ADR-005 — Seguridad para portafolio: Multi-tenant + Cuenta Demo

**Decisión:** Implementar **multi-tenant** y un **tenant demo** público y limitado.

**Motivo:**

- El proyecto se expondrá en internet; sin protección sería vulnerable a spam y vandalismo.
- Multi-tenant permite cuentas reales con datos privados y una demo aislada.
- Muestra habilidades avanzadas (aislamiento por tenant), muy valoradas.

**Alternativas consideradas:**

- Sitio público sin auth (alto riesgo de abuso).
- Login único (una sola cuenta global) (no separa datos y es menos “real”).
- API key simple (más rápido, pero menos “producto real”).

**Consecuencias:**

- Todas las tablas principales incluyen `tenant_id`.
- Todas las consultas deben filtrar por `tenant_id`.
- Se requiere autenticación y autorización.

---

## ADR-006 — Estrategia de autenticación: JWT (access token)

**Decisión:** Usar **JWT** con `Authorization: Bearer <token>`.

**Motivo:**

- Simple para API REST.
- Muy común en backend Node.
- Fácil de consumir desde cualquier frontend.

**Alternativas consideradas:**

- Sesiones server-side (más infraestructura y manejo).
- Cookies HttpOnly (muy buena opción, pero requiere configuración CORS más cuidadosa; se puede agregar después).

**Consecuencias:**

- El cliente debe almacenar el token (en portafolio: idealmente HttpOnly en frontend real, pero para MVP puede ser local).
- Se debe implementar expiración y manejo de errores 401.

---

## ADR-007 — Privacidad y datos ficticios

**Decisión:** El tenant demo usará **datos ficticios** y nunca datos reales.

**Motivo:**

- Buenas prácticas y ética profesional.
- Evita riesgos legales/privacidad al exponer el proyecto públicamente.

**Consecuencias:**

- Se debe tener un seed de demo con datos inventados.
- Si se hace reset demo, debe recrear datos de muestra.

---

## ADR-008 — Medidas anti-abuso en demo (MVP)

**Decisión:** Proteger la demo con al menos una medida:

- rate limit y/o cuotas y/o reset.

**Motivo:**

- Evitar spam, abuso de endpoints y basura de datos.

**Alternativas consideradas:**

- Dejar demo sin límites (mala idea en internet).

**Consecuencias:**

- Agregar configuración de rate limit o límites por cantidad.
- Posible endpoint `/demo/reset` (protegido) o reset programado.

---

## ADR-009 — Roadmap: versión Node primero, luego Django

**Decisión:** Implementar primero en **NestJS**, luego replicar en **Django**.

**Motivo:**

- Alineación con mercado laboral (Node/Nest + TS).
- Comparación real de stacks sobre el mismo dominio (muy buen material para portafolio).

**Consecuencias:**

- Mantener documentación como “fuente de verdad” para ambos stacks.
- El modelo de datos y endpoints deben mantenerse consistentes.

---

## Próximas decisiones (pendientes)

- [ ] ORM/migrations: Prisma vs TypeORM (o alternativa).
- [ ] ¿Registro abierto o por invitación?
- [ ] Demo: ¿escritura limitada o solo lectura?
- [ ] Estrategia de refresh token (si se necesita).

---

Si quieres, el siguiente paso ya es **pasar a implementación** con orden:

1. Elegir ORM (**Prisma** o **TypeORM**)
2. Crear estructura del repo
3. `docker-compose.yml` (api + db + volumen)
4. Generar proyecto NestJS
5. Implementar Auth + tenant guard
6. Luego módulos: workers → cases → actions

¿Quieres que usemos **Prisma** (muy popular y simple) o **TypeORM** (clásico en Nest)?
