Aquí tienes el **plan modificado** incorporando lo nuevo: **multi-tenant + cuenta demo pública + cuentas reales privadas** (para que puedas mostrarlo en internet sin que te lo destruyan).

---

# Plan actualizado del proyecto

✅ **Primero lo hacemos en Node (NestJS + TypeScript)** con documentación y enfoque “empresa real”.
✅ Luego haces **la misma solución en Django** (comparación de stacks).
✅ La app estará expuesta en internet, por lo que incluimos **autenticación + aislamiento de datos por cuenta (tenant)** y un **modo demo** seguro.

---

## 1) Documento base del proyecto

### 1.1 Propósito

**Sistema:** Gestor de Casos Sociales / Bienestar
**Objetivo:** registrar, priorizar, hacer seguimiento y cerrar casos de trabajadores con trazabilidad.
**Modo portafolio:** permitir que cualquiera pruebe el sistema **sin afectar** datos reales (demo), y que usuarios reales tengan datos **privados**.

### 1.2 Alcance MVP (versión 1)

#### Núcleo funcional

- CRUD de **Trabajadores**
- CRUD de **Casos**
- CRUD de **Acciones/Seguimientos**
- Filtros: estado, prioridad, tema, sucursal, fechas
- Regla: _no se puede cerrar un caso sin al menos 1 acción_
- **Logs** (consola + archivo) y auditoría básica

#### Seguridad / Portafolio (nuevo, MVP)

- **Autenticación** (login)
- **Registro** para cuentas reales (opcional si quieres cerrarlo inicialmente)
- **Aislamiento de datos por cuenta** (multi-tenant)
- **Cuenta Demo** pública (cualquiera puede entrar) con:
  - datos ficticios
  - límites (cuotas y/o rate limit)
  - opción de “reset demo” (manual o automático)

### 1.3 No entra aún (versión 2)

- relatos largos “tipo nota” y adjuntos (PDF, imágenes)
- reportes complejos / dashboards
- multi-rol granular (jefaturas, permisos finos)
- notificaciones (correo/WhatsApp/push)
- integraciones con otros sistemas

---

## 2) Arquitectura con Node

Como empresa (y para portafolio), lo más ordenado es:

- **API REST NestJS** (lógica de negocio)
- **PostgreSQL** (datos)
- **Docker Compose** (infra local)
- Frontend después (o Postman al inicio)

### Opción recomendada

- **NestJS + TypeScript** (estructura sólida, escalable, muy “empresa”)

---

## 3) Modelo de datos (MVP) — actualizado con multi-tenant

### Entidades base (dominio)

- **trabajador**
  - id, tenant_id, rut, nombre, correo?, telefono?, sucursal, area?, activo, timestamps

- **caso**
  - id, tenant_id, trabajador_id, tema, prioridad, estado, canal_ingreso, descripcion_breve, fecha_apertura, fecha_cierre?, timestamps

- **accion_caso**
  - id, tenant_id, caso_id, fecha_accion, tipo_accion, descripcion, proxima_revision?, timestamps

- **audit_log** (opcional MVP, recomendado)
  - id, tenant_id, actor_user_id, accion, entidad, entidad_id, detalle, fecha

### Entidades de seguridad (nuevo)

- **tenant** (espacio de datos)
  - id, name, type(demo|real), created_at

- **user**
  - id, tenant_id, email, password_hash, role(admin|user), active, timestamps

### Enums (catálogos)

- tema: salud | familiar | endeudamiento | inclusion | otro
- prioridad: alta | media | baja
- estado: abierto | seguimiento | cerrado
- tipo_accion: llamada | correo | reunion | derivacion | gestion | otro

📌 **Regla clave:** toda lectura/escritura filtra por `tenant_id` del usuario autenticado.

---

## 4) Endpoints que documentamos (API contract) — actualizado

### Auth (nuevo)

- `POST /auth/demo` → entra con “cuenta demo”
- `POST /auth/register` → crea cuenta real (opcional si lo dejas abierto)
- `POST /auth/login`
- `GET /auth/me`
- `POST /auth/logout` (opcional, según estrategia)

### Trabajadores

- `POST /workers`
- `GET /workers`
- `GET /workers/:id`
- `PATCH /workers/:id`

### Casos

- `POST /cases`
- `GET /cases` (filtros por query)
- `GET /cases/:id`
- `PATCH /cases/:id`
- `POST /cases/:id/close`

### Acciones

- `POST /cases/:id/actions`
- `GET /cases/:id/actions`

### Demo (opcional MVP, recomendado)

- `POST /demo/reset` (protegido, o solo admin)
  o programar reset automático.

**Filtros recomendados** en `GET /cases`:

- `status`, `priority`, `topic`, `branch`, `from`, `to`, `q`

---

## 5) Estándares “empresa” desde el inicio — actualizado

### Variables de entorno (Node)

- `NODE_ENV=development|production`
- `PORT=3000`
- `DATABASE_URL=postgresql://user:pass@db:5432/gc_bienestar`
- `JWT_SECRET=...`
- `JWT_EXPIRES_IN=...`
- `LOG_LEVEL=info|debug|error`
- `DEMO_TENANT_ID=...` (si lo fijas) o `DEMO_MODE=true`
- `DEMO_MAX_CASES=...` (cuotas)
- `RATE_LIMIT_WINDOW=...` / `RATE_LIMIT_MAX=...` (si aplicas)

### Logs

- dev: consola
- prod: consola + archivo (o JSON)

### Validación

- DTOs + mensajes claros
- errores estandarizados (404, 400, 401, 403)

### Testing mínimo (después)

- 2–3 tests de endpoints críticos
- test regla: “no cerrar sin acción”
- test aislamiento: “no leer datos de otro tenant”

---

## 6) Estructura de repo (para documentar) — actualizado

```
/backend-node
  /src
    /modules
      /auth
      /tenants
      /users
      /workers
      /cases
      /actions
    /common
      /guards
      /filters
      /interceptors
  docker-compose.yml
  .env.example
  README.md
  docs/
    01-requisitos.md
    02-modelo-datos.md
    03-endpoints.md
    04-decisions-ADR.md
```

---

## 7) Paquete de documentación mínimo (igual, pero con multi-tenant)

1. **README.md** → qué es, cómo correrlo, comandos, env
2. **docs/01-requisitos.md** → alcance + seguridad demo + reglas
3. **docs/02-modelo-datos.md** → tablas + enums + tenant_id
4. **docs/03-endpoints.md** → endpoints + ejemplos + auth
5. **docs/04-decisions-ADR.md** → decisiones (Nest, multi-tenant, demo, etc.)

---

## Siguiente paso (sin saltarnos nada)

Yo te armo la documentación base lista para copiar/pegar en este orden:

1. `docs/01-requisitos.md` (actualizado con demo + tenants)
2. `docs/02-modelo-datos.md`
3. `docs/03-endpoints.md`
4. README

Y recién después: Docker Compose + proyecto NestJS.

---

Si quieres, el próximo mensaje hago **docs/02-modelo-datos.md** completo (con tablas, claves, relaciones y enums) ya con el enfoque multi-tenant + demo.
