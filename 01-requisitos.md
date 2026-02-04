# docs/01-requisitos.md — Requisitos del sistema (MVP) — Actualizado con Demo + Multi-tenant

## 1. Objetivo

Construir un sistema web para el área de **Bienestar / Trabajo Social** que permita **registrar, priorizar, hacer seguimiento y cerrar** casos sociales de trabajadores/as, manteniendo **trazabilidad** de acciones realizadas.

El sistema se publicará en internet como **proyecto de portafolio**, por lo que debe incluir un **modo demo** para que cualquier persona pueda probarlo sin afectar datos reales, y un modo **cuenta real** donde cada usuario vea solo sus propios datos.

---

## 2. Usuarios del sistema

### 2.1 Visitante (Demo pública)

- Puede acceder a una **Cuenta Demo** para probar el sistema.
- La demo usa **datos ficticios** y está **limitada** para evitar abuso (spam).

### 2.2 Usuario registrado (Cuenta real)

- Puede crear y administrar sus propios datos privados.
- Solo puede acceder a datos asociados a su cuenta/espacio (**tenant**).

### 2.3 Administrador (MVP)

- Puede gestionar todo dentro de su tenant.
- (Opcional) Puede ejecutar un **reset de demo** si se habilita endpoint/admin interno.

---

## 3. Alcance MVP (versión 1)

### 3.1 Autenticación y acceso (MVP)

- Login de usuario (cuentas reales).
- Opción “Entrar como Demo” (sin registro).
- El sistema debe emitir un token de sesión (por ejemplo JWT).
- Toda operación de lectura/escritura debe estar asociada a un **tenant**.

**Multi-tenant (MVP):**

- Cada usuario pertenece a un `tenant`.
- Todas las entidades principales (workers, cases, actions) están asociadas a `tenant_id`.
- Un usuario **no puede leer ni modificar** datos de otro tenant.

### 3.2 Gestión de trabajadores/as

- Crear trabajador/a.
- Actualizar trabajador/a.
- Listar y buscar trabajadores/as.
- (Opcional MVP) Ver detalle de trabajador/a.

**Datos mínimos:**

- RUT (único por tenant)
- Nombre
- Sucursal (opcional)
- Estado activo/inactivo

### 3.3 Gestión de casos

- Crear caso asociado a un trabajador/a.
- Listar casos con filtros.
- Ver detalle de caso.
- Actualizar caso (estado, prioridad, tema, descripción breve).
- Cerrar caso con regla de negocio.

**Datos mínimos:**

- Trabajador/a asociado
- Tema
- Prioridad
- Estado
- Descripción breve (relato corto)
- Fecha apertura / fecha cierre

### 3.4 Seguimiento / acciones del caso

- Registrar acciones (seguimientos) por caso.
- Listar acciones por caso.

**Datos mínimos:**

- Tipo de acción
- Descripción
- Fecha de acción
- Próxima revisión (opcional)

### 3.5 Auditoría básica (recomendado en MVP)

- Registrar eventos relevantes: creación/edición/cierre de casos y creación de acciones.
- Objetivo: trazabilidad mínima y debugging.

### 3.6 Filtros (MVP)

En listados (principalmente casos), permitir:

- por `status` (estado)
- por `priority` (prioridad)
- por `topic` (tema)
- por `branch` (sucursal)
- por rango de fechas (`from` / `to`)
- búsqueda simple (`q`) en descripción o nombre

---

## 4. Reglas de negocio (MVP)

1. **Un caso siempre pertenece a un trabajador/a.**
2. **No se puede cerrar un caso sin al menos 1 acción registrada.**
3. Todo dato creado debe quedar asociado a un **tenant**.
4. Un usuario solo puede operar dentro de su **tenant**.
5. El **RUT** debe ser único dentro del tenant.
6. Al cerrar un caso, se debe guardar **fecha de cierre**.
7. Los campos categóricos (tema, prioridad, estado, tipo de acción) deben validarse contra listas permitidas (enums/catálogos).

---

## 5. Requisitos del modo Demo (MVP)

### 5.1 Aislamiento de demo

- La demo opera dentro de un tenant tipo `demo`.
- Los datos de demo **no se mezclan** con datos de cuentas reales.

### 5.2 Límites anti-abuso (MVP)

La demo debe tener al menos una de estas medidas:

- **Rate limit** por IP en endpoints de escritura, y/o
- **Cuotas** (máximo de casos/acciones), y/o
- **Reset** del tenant demo (manual o automático).

### 5.3 Datos ficticios

- Los datos del tenant demo son ficticios (nunca datos reales de personas).

---

## 6. Requisitos no funcionales (mínimos)

- **Persistencia:** PostgreSQL con volumen Docker en desarrollo.
- **Configuración por variables de entorno:** DB, puertos, secrets, modo demo.
- **Logging mínimo:** errores y eventos principales (dev: consola; prod: consola + archivo o JSON).
- **Estructura profesional:** módulos separados y documentación en `docs/`.
- **Seguridad base:** hash de contraseña, validación de entradas, respuestas de error consistentes.

---

## 7. Fuera de alcance (por ahora)

- Relatos largos y adjuntos (PDF/imágenes).
- Reportería avanzada / dashboards.
- Permisos granulares (roles finos, multi-área, jefaturas).
- Notificaciones (correo/WhatsApp/push).
- Integraciones externas (tickets/intranet/ERP).
- Cifrado de campos sensibles a nivel de BD (se puede agregar después).

---

## 8. Checklist de decisiones pendientes

- [ ] ¿La demo permitirá **escritura** o será **solo lectura**?
- [ ] ¿El registro `/auth/register` estará abierto al público o será solo por invitación?
- [ ] ¿Reset demo manual, automático (diario), o ambos?
- [ ] Límite de caracteres para “descripción breve” (ej. 280–500).

---
