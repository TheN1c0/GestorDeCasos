Perfecto. Entonces aquí va el **docs/02-modelo-datos.md** completo, ya con **multi-tenant + demo** y pensado para NestJS + PostgreSQL.

---

# docs/02-modelo-datos.md — Modelo de datos (MVP)

## 1. Objetivo del modelo

Definir las entidades necesarias para:

- gestionar **casos sociales** y sus **acciones/seguimientos**
- asociar todo a un **trabajador/a**
- separar datos por **tenant** (cuenta/organización)
- permitir una **cuenta demo** pública con datos aislados
- soportar autenticación con **usuarios**

---

## 2. Principio clave: Multi-tenant

Toda entidad del dominio (workers, cases, actions, audit logs) incluye:

- `tenant_id` (FK a `tenants.id`)

📌 Regla: **un usuario solo puede leer/escribir datos de su propio tenant**.

---

## 3. Tablas

### 3.1 `tenants`

Representa el “espacio de datos” (empresa/cuenta). Incluye un tenant especial tipo `demo`.

**Campos**

- `id` (PK, UUID o serial)
- `name` (texto, requerido)
- `type` (enum: `demo` | `real`)
- `created_at` (timestamp)
- `updated_at` (timestamp)

**Reglas**

- Debe existir al menos 1 tenant `demo`.
- Los tenants reales se crean por registro (si habilitas registro).

---

### 3.2 `users`

Usuarios que inician sesión y pertenecen a un tenant.

**Campos**

- `id` (PK)
- `tenant_id` (FK → tenants.id, requerido)
- `email` (texto, requerido, único global recomendado)
- `password_hash` (texto, requerido)
- `role` (enum: `admin` | `user`) _(MVP: puede ser solo admin)_
- `is_active` (boolean, default true)
- `created_at`, `updated_at` (timestamp)

**Reglas**

- No se guarda contraseña en texto plano, solo `password_hash`.
- En MVP puedes crear 1 usuario admin por tenant.
- Para demo: puedes tener un usuario demo fijo o un endpoint `/auth/demo`.

---

### 3.3 `workers`

Trabajadores/as gestionados por el equipo.

**Campos**

- `id` (PK)
- `tenant_id` (FK → tenants.id, requerido)
- `rut` (texto, requerido) _(único por tenant)_
- `full_name` (texto, requerido)
- `email` (texto, opcional)
- `phone` (texto, opcional)
- `branch` (texto, opcional) _(sucursal)_
- `area` (texto, opcional)
- `is_active` (boolean, default true)
- `created_at`, `updated_at` (timestamp)

**Reglas**

- `rut` debe ser único **dentro del tenant**.
- Un worker puede tener múltiples casos.

---

### 3.4 `cases`

Caso social asociado a un worker.

**Campos**

- `id` (PK)
- `tenant_id` (FK → tenants.id, requerido)
- `worker_id` (FK → workers.id, requerido)
- `topic` (enum, requerido)
- `priority` (enum, requerido)
- `status` (enum, requerido)
- `intake_channel` (enum o texto, opcional) _(canal ingreso)_
- `short_description` (texto, requerido) _(relato corto)_
- `opened_at` (timestamp, requerido)
- `closed_at` (timestamp, opcional)
- `created_at`, `updated_at` (timestamp)

**Reglas**

- Un caso pertenece a un worker y a un tenant.
- Si `status = closed` → `closed_at` debe tener valor.
- No se puede cerrar un caso si no tiene al menos 1 acción.

---

### 3.5 `case_actions`

Acciones o seguimientos de un caso.

**Campos**

- `id` (PK)
- `tenant_id` (FK → tenants.id, requerido)
- `case_id` (FK → cases.id, requerido)
- `action_type` (enum, requerido)
- `description` (texto, requerido)
- `action_date` (timestamp, requerido)
- `next_review_date` (timestamp, opcional)
- `created_at`, `updated_at` (timestamp)

**Reglas**

- Un caso puede tener 0..N acciones.
- Para cerrar caso: mínimo 1 acción.

---

### 3.6 `audit_logs` (opcional MVP, recomendado)

Trazabilidad mínima de acciones relevantes.

**Campos**

- `id` (PK)
- `tenant_id` (FK → tenants.id, requerido)
- `actor_user_id` (FK → users.id, opcional o requerido según implementación)
- `event_type` (texto/enum: `CASE_CREATED`, `CASE_UPDATED`, `CASE_CLOSED`, `ACTION_CREATED`, etc.)
- `entity_type` (texto: `case`, `worker`, `case_action`)
- `entity_id` (texto/id)
- `detail` (json/texto, opcional)
- `created_at` (timestamp)

---

## 4. Catálogos (Enums)

### 4.1 `topic` (tema)

- `health`
- `family`
- `debt`
- `inclusion`
- `other`

_(Puedes cambiar los valores a español en presentación, pero en DB conviene en inglés para consistencia.)_

### 4.2 `priority`

- `high`
- `medium`
- `low`

### 4.3 `status`

- `open`
- `follow_up`
- `closed`

### 4.4 `action_type`

- `call`
- `email`
- `meeting`
- `referral`
- `management`
- `other`

### 4.5 `tenant.type`

- `demo`
- `real`

### 4.6 `user.role`

- `admin`
- `user`

---

## 5. Relaciones (resumen)

- `tenants 1 ── N users`
- `tenants 1 ── N workers`
- `tenants 1 ── N cases`
- `tenants 1 ── N case_actions`
- `workers 1 ── N cases`
- `cases 1 ── N case_actions`

---

## 6. Índices y unicidad recomendada

- `users.email` → UNIQUE (global)
- `workers (tenant_id, rut)` → UNIQUE (por tenant)
- `cases (tenant_id, worker_id)` → INDEX (búsquedas)
- `case_actions (tenant_id, case_id)` → INDEX

---

## 7. Consideraciones para el modo demo

Opciones de implementación (MVP elige 1):

1. **Tenant demo fijo**
   - Existe un tenant `demo` con id conocido
   - `/auth/demo` entrega un token para ese tenant

2. **Usuario demo fijo**
   - usuario `demo@demo.cl` con password simple (pero cuidado si será público)

Medidas anti-spam (recomendadas):

- Rate limit por IP en endpoints de escritura (demo)
- Cuotas (máximo N casos demo, N acciones, etc.)
- Reset manual o automático del tenant demo

---

## 8. Pendientes para decidir (rápido)

- [ ] ¿Usaremos IDs UUID o autoincrementales?
- [ ] ¿El registro `/auth/register` estará abierto o solo tú creas cuentas?
- [ ] ¿El demo permitirá escritura o será solo lectura?

---

Cuando quieras, sigo con el **docs/03-endpoints.md** (con ejemplos de request/response y reglas de auth/tenant).
