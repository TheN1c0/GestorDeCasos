# docs/03-endpoints.md — Contrato de API (MVP) — NestJS + Multi-tenant + Demo

## 0. Convenciones generales

### Base URL

- Local: `http://localhost:3000`
- Prefijo sugerido: `/api` (opcional). En este documento asumo **sin prefijo** para simplificar.

### Autenticación

- Usaremos **JWT** en header:
  - `Authorization: Bearer <token>`

### Multi-tenant (regla obligatoria)

- El `tenant_id` **no se envía** en el body desde el cliente.
- El backend lo obtiene desde el **token** (`req.user.tenantId`) y filtra automáticamente.

### Respuestas de error (forma sugerida)

- `400` Validación
- `401` No autenticado
- `403` No autorizado (tenant/rol)
- `404` No encontrado (o no pertenece al tenant)
- `409` Conflicto (ej. RUT duplicado)

---

## 1. Auth

### 1.1 Entrar como Demo

**POST** `/auth/demo`

**Descripción:** entrega un token para el tenant `demo`.
**Body:** vacío

**Response 200**

```json
{
  "accessToken": "jwt...",
  "user": {
    "id": "demo-user-id",
    "email": "demo@demo.local",
    "role": "admin",
    "tenantId": "demo-tenant-id",
    "tenantType": "demo"
  }
}
```

---

### 1.2 Registro (Cuenta real) _(si se habilita)_

**POST** `/auth/register`

**Body**

```json
{
  "tenantName": "Mi cuenta",
  "email": "user@example.com",
  "password": "UnaClaveLarga123!"
}
```

**Response 201**

```json
{
  "accessToken": "jwt...",
  "user": {
    "id": "user-id",
    "email": "user@example.com",
    "role": "admin",
    "tenantId": "tenant-id",
    "tenantType": "real"
  }
}
```

**Errores**

- `409` si el email ya existe

---

### 1.3 Login

**POST** `/auth/login`

**Body**

```json
{
  "email": "user@example.com",
  "password": "UnaClaveLarga123!"
}
```

**Response 200**

```json
{
  "accessToken": "jwt...",
  "user": {
    "id": "user-id",
    "email": "user@example.com",
    "role": "admin",
    "tenantId": "tenant-id",
    "tenantType": "real"
  }
}
```

**Errores**

- `401` credenciales inválidas

---

### 1.4 Perfil del usuario actual

**GET** `/auth/me`

**Headers**

- `Authorization: Bearer <token>`

**Response 200**

```json
{
  "id": "user-id",
  "email": "user@example.com",
  "role": "admin",
  "tenantId": "tenant-id",
  "tenantType": "real"
}
```

---

### 1.5 Logout _(opcional)_

**POST** `/auth/logout`

**Nota:** con JWT “stateless” el logout suele ser del lado del cliente (borrar token).
Si implementas refresh tokens, aquí se invalida el refresh.

---

## 2. Workers (Trabajadores/as)

> Todos estos endpoints requieren `Authorization`.

### 2.1 Crear worker

**POST** `/workers`

**Body**

```json
{
  "rut": "12.345.678-9",
  "fullName": "Juan Pérez",
  "email": "juan@example.com",
  "phone": "+56912345678",
  "branch": "Santiago",
  "area": "Operaciones",
  "isActive": true
}
```

**Response 201**

```json
{
  "id": "worker-id",
  "rut": "12.345.678-9",
  "fullName": "Juan Pérez",
  "email": "juan@example.com",
  "phone": "+56912345678",
  "branch": "Santiago",
  "area": "Operaciones",
  "isActive": true,
  "createdAt": "2026-02-02T12:00:00.000Z",
  "updatedAt": "2026-02-02T12:00:00.000Z"
}
```

**Errores**

- `409` si `rut` ya existe en el tenant
- `400` validación

---

### 2.2 Listar workers (con búsqueda)

**GET** `/workers?q=juan&branch=Santiago&isActive=true&page=1&pageSize=20`

**Response 200**

```json
{
  "items": [
    {
      "id": "worker-id",
      "rut": "12.345.678-9",
      "fullName": "Juan Pérez",
      "branch": "Santiago",
      "isActive": true
    }
  ],
  "page": 1,
  "pageSize": 20,
  "total": 1
}
```

---

### 2.3 Obtener detalle de worker

**GET** `/workers/:id`

**Response 200** (mismo formato completo del create)

**Errores**

- `404` si no existe o no pertenece al tenant

---

### 2.4 Actualizar worker

**PATCH** `/workers/:id`

**Body (parcial)**

```json
{
  "phone": "+56900000000",
  "isActive": false
}
```

**Response 200** (worker actualizado)

---

## 3. Cases (Casos)

> Todos estos endpoints requieren `Authorization`.

### 3.1 Crear caso

**POST** `/cases`

**Body**

```json
{
  "workerId": "worker-id",
  "topic": "health",
  "priority": "high",
  "status": "open",
  "intakeChannel": "email",
  "shortDescription": "Relato breve del caso (MVP).",
  "openedAt": "2026-02-02T12:00:00.000Z"
}
```

**Response 201**

```json
{
  "id": "case-id",
  "workerId": "worker-id",
  "topic": "health",
  "priority": "high",
  "status": "open",
  "intakeChannel": "email",
  "shortDescription": "Relato breve del caso (MVP).",
  "openedAt": "2026-02-02T12:00:00.000Z",
  "closedAt": null,
  "createdAt": "2026-02-02T12:01:00.000Z",
  "updatedAt": "2026-02-02T12:01:00.000Z"
}
```

**Errores**

- `404` si `workerId` no existe en el tenant
- `400` validación de enums/fechas

---

### 3.2 Listar casos con filtros

**GET** `/cases?status=open&priority=high&topic=health&branch=Santiago&from=2026-02-01&to=2026-02-28&q=relato&page=1&pageSize=20`

**Response 200**

```json
{
  "items": [
    {
      "id": "case-id",
      "workerId": "worker-id",
      "workerName": "Juan Pérez",
      "topic": "health",
      "priority": "high",
      "status": "open",
      "openedAt": "2026-02-02T12:00:00.000Z",
      "closedAt": null
    }
  ],
  "page": 1,
  "pageSize": 20,
  "total": 1
}
```

---

### 3.3 Detalle de caso

**GET** `/cases/:id`

**Response 200**

```json
{
  "id": "case-id",
  "worker": {
    "id": "worker-id",
    "rut": "12.345.678-9",
    "fullName": "Juan Pérez",
    "branch": "Santiago"
  },
  "topic": "health",
  "priority": "high",
  "status": "open",
  "intakeChannel": "email",
  "shortDescription": "Relato breve del caso (MVP).",
  "openedAt": "2026-02-02T12:00:00.000Z",
  "closedAt": null,
  "createdAt": "2026-02-02T12:01:00.000Z",
  "updatedAt": "2026-02-02T12:01:00.000Z"
}
```

**Errores**

- `404` si no existe o no pertenece al tenant

---

### 3.4 Actualizar caso

**PATCH** `/cases/:id`

**Body (parcial)**

```json
{
  "priority": "medium",
  "status": "follow_up",
  "shortDescription": "Actualización breve."
}
```

**Response 200** (caso actualizado)

**Errores**

- `400` si enums inválidos

---

### 3.5 Cerrar caso (regla: requiere al menos 1 acción)

**POST** `/cases/:id/close`

**Body (opcional)**

```json
{
  "closedAt": "2026-02-10T18:00:00.000Z"
}
```

**Response 200**

```json
{
  "id": "case-id",
  "status": "closed",
  "closedAt": "2026-02-10T18:00:00.000Z"
}
```

**Errores**

- `409` si el caso no tiene acciones registradas

```json
{
  "message": "Cannot close case without at least one action."
}
```

---

## 4. Case Actions (Acciones)

> Todos estos endpoints requieren `Authorization`.

### 4.1 Crear acción para un caso

**POST** `/cases/:id/actions`

**Body**

```json
{
  "actionType": "call",
  "description": "Se realizó llamada y se acordó seguimiento.",
  "actionDate": "2026-02-03T09:00:00.000Z",
  "nextReviewDate": "2026-02-10T09:00:00.000Z"
}
```

**Response 201**

```json
{
  "id": "action-id",
  "caseId": "case-id",
  "actionType": "call",
  "description": "Se realizó llamada y se acordó seguimiento.",
  "actionDate": "2026-02-03T09:00:00.000Z",
  "nextReviewDate": "2026-02-10T09:00:00.000Z",
  "createdAt": "2026-02-03T09:01:00.000Z",
  "updatedAt": "2026-02-03T09:01:00.000Z"
}
```

**Errores**

- `404` si el caso no existe o no pertenece al tenant

---

### 4.2 Listar acciones de un caso

**GET** `/cases/:id/actions?page=1&pageSize=50`

**Response 200**

```json
{
  "items": [
    {
      "id": "action-id",
      "caseId": "case-id",
      "actionType": "call",
      "description": "Se realizó llamada...",
      "actionDate": "2026-02-03T09:00:00.000Z",
      "nextReviewDate": "2026-02-10T09:00:00.000Z"
    }
  ],
  "page": 1,
  "pageSize": 50,
  "total": 1
}
```

---

## 5. Demo (opcional MVP, recomendado)

### 5.1 Reset demo

**POST** `/demo/reset`

**Descripción:** borra y re-seedea datos del tenant demo.
**Acceso recomendado:** solo `admin` del tenant demo, o una clave interna de admin.

**Response 200**

```json
{
  "ok": true
}
```

---

## 6. Reglas de seguridad a implementar (checklist)

- [ ] Guard (auth) obligatorio en endpoints de dominio.
- [ ] Filtro por `tenantId` en todas las queries.
- [ ] Si `:id` existe pero es de otro tenant → responder `404` (no filtrar info).
- [ ] Rate limit en escritura para demo (si estará público).
- [ ] Validación DTO para enums y campos requeridos.

---

## 7. Enums (recordatorio)

- `topic`: `health | family | debt | inclusion | other`
- `priority`: `high | medium | low`
- `status`: `open | follow_up | closed`
- `actionType`: `call | email | meeting | referral | management | other`

---

Si quieres, lo siguiente lógico es **docs/04-decisions-ADR.md** (decisiones técnicas + por qué multi-tenant + por qué demo) y luego recién pasamos a **Docker Compose + scaffolding de NestJS**.
