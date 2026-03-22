# 📋 Backend — solicitudes PANDORA (desde Front)

---

## 🏛️ Arquitectura y Super Admin

Cambios para usuarios de alto nivel y entidad **Super Admin**: jerarquía de privilegios y **DTO de creación reducido**.

### `createSuperAdmin` — parámetros requeridos

| Campo | Rol |
|--------|-----|
| **email** | Acceso único |
| **password** | Hash en servidor antes de guardar |
| **razonSocialId** | Vincular a la razón social (API: `razon_social_id`) |

**Backend:** refactorizar `POST /users/super-admin` para aceptar **solo** esos campos (quitar obligatoriedad de nombre/apellido en el alta; el perfil se completa después si aplica).

---

## 👥 Quién crea a quién

| Rol | Crea | Alcance |
|-----|------|---------|
| **SUPER_ADMIN** | Otros Super Admins | Empresa propietaria / marco acordado |
| **SUPER_ADMIN** | Admins de empresa | Propia organización **y** empresas cliente |
| **SUPER_ADMIN** | Operativos | **Solo** de la **empresa propietaria** |
| **ADMIN_EMPRESA** | Otros admins y operativos | **Solo** su empresa (su tenant) |

**Backend:** reglas en guards / servicios de `POST /users/*` coherentes con la tabla.

---

## 📦 Planes y ventanas del menú

- Las **ventanas** del administrador de empresa **dependen del plan** y de la **empresa**: si contratan más alcance o se **habilita manualmente** más módulos, la lista efectiva debe reflejarlo (plan + extras acordados).
- **SUPER_ADMIN** ve todo el menú (sin filtrar por plan).
- **ADMIN_EMPRESA** se filtra según módulos efectivos (ver `plan_modulos`).

**Claves de módulo** (menú lateral en front):

| Clave | Área |
|--------|------|
| `facturacion` | Facturación |
| `ventas_pos` | POS / ventas |
| `maestros` | Personas, productos, categorías |
| `contabilidad` | Contabilidad |
| `reportes_financieros` | Reportes |
| `tesoreria` | Tesorería |
| `inventario` | Inventario |
| `activos_fijos` | Activos fijos |
| `rrhh` | RRHH |
| `crm` | CRM / preventa |
| `estructura` | Establecimientos, emisión, secuenciales |

Planes en BD deben poder asociar un subconjunto de estas claves; los extras manuales se **suman** a la lista que resuelve el login.

---

## 🏢 Config empresa · avatar · logo

**Cambios backend (no narración de “hoy pasa X”):**

1. **`PATCH /companies/:razonSocialId/config`** — DTO: agregar opcionales **`nombre_comercial`**, **`direccion_matriz`**; persistir en **`razones_sociales`** (y sincronizar `tenants` si aplica).
2. **`PATCH /companies/:razonSocialId/logo`** — multipart; validar que la respuesta permita al front mostrar el logo.
3. **Perfil usuario** — `PATCH /users/profile/complete` + campos en sesión para avatar/datos; URLs estables y accesibles (CORS si aplica).

---

## 🔐 Login / sesión y `plan_modulos`

**Qué debe hacer el backend**

- Incluir en **`empresa_actual`** el array **`plan_modulos`** (strings de la tabla de arriba). Mismo criterio al listar **`mis_empresas`** si el usuario cambia de empresa.
- **Reglas que usa el front:**
  - Sin `plan_modulos`, `null` o `[]` → muestra todos los módulos (transición).
  - Con valores → el **admin de empresa** solo ve esas secciones; **super admin** ignora el filtro.
  - Overrides contractuales → deben entrar en la lista **ya resuelta** que devuelve el login.

---

## 🛠️ Qué crear, qué modificar (interés backend)

### DTOs / validación

| Qué | Acción |
|-----|--------|
| `PatchCompanyConfigDto` (o equivalente) | Añadir **`nombre_comercial`**, **`direccion_matriz`** opcionales; aplicar en servicio/Prisma. |
| Body `POST /users/super-admin` | Solo **`email`**, **`password`**, **`razon_social_id`**. |
| Respuesta `POST /auth/login` | Añadir **`plan_modulos`** en `empresa_actual` (y coherente en `mis_empresas` si aplica). |

### Endpoints a tener listos o a cerrar contrato

| Ruta | Nota |
|------|------|
| `PATCH /tenants/:id/subscription/plan` | Body `{ plan_id }`; coherente con planes ↔ módulos |
| `PATCH /tenants/:id/subscription/vigencia` | Fechas / modalidad de suscripción |
| `PATCH /tenants/:id/estado` | Activo / inactivo (si el front ya lo llama) |
| `DELETE /tenants/:id` | Baja tenant (si aplica) |

*(El resto de rutas de usuarios, empresas, planes y firma electrónica: validar permisos por rol y cuerpos esperados por el front; no se listan aquí como “catálogo consumido”, sino para alinear implementación.)*

---

## ⚖️ Plan vs RBAC

- **`plan_modulos`** = qué **bloques de menú** ve el admin de empresa.
- **Permisos finos** (`rol.permisos`, etc.) = acciones dentro de pantallas; cuando estén estables, el front puede usarlos.

---

## ✅ Checklist rápido

- [ ] `POST /users/super-admin` reducido a email + password + `razon_social_id`
- [ ] Reglas de creación de usuarios (tabla jerarquía)
- [ ] `plan_modulos` en login + fusión plan + habilitaciones manuales
- [ ] `PATCH .../config` con nombre comercial y dirección matriz
- [ ] Suscripción plan / vigencia operativos
