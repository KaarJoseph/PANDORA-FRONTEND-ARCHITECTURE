# Backend PANDORA — cambios y mejoras 

---

## 1. Arquitectura de base de datos y lógica de Super Admin

Cambios para gestión de usuarios de alto nivel y creación de Super Admin: jerarquía de privilegios y **DTO de inicialización reducido**.

### 1.1 `createSuperAdmin` — parámetros de entrada

Refactorizar el alta de Super Admins: solo identidad y vínculo empresarial.

| Parámetro | Obligatorio | Notas |
|-----------|-------------|--------|
| `email` | Sí | Acceso único |
| `password` | Sí | Persistir solo hash |
| `razonSocialId` / `razon_social_id` | Sí | Vinculación a la razón social (entidad legal) |

**Cambio en backend:** `POST /users/super-admin` acepta únicamente esos campos; nombre/apellido vía `PATCH /users/profile/complete` u otro flujo posterior si aplica.

---

## 2. Jerarquía de creación de usuarios

| Actor | Puede crear | Restricción |
|-------|-------------|-------------|
| `SUPER_ADMIN` | Otros `SUPER_ADMIN` | Según reglas de la empresa propietaria / grupo |
| `SUPER_ADMIN` | `ADMIN_EMPRESA` | Cualquier empresa (propia y clientes) |
| `SUPER_ADMIN` | `OPERATIVO` | Solo tenant de la **empresa propietaria** |
| `ADMIN_EMPRESA` | `ADMIN_EMPRESA` y `OPERATIVO` | Solo usuarios de **su** tenant |

**Cambio en backend:** validar en servicio/guards en `POST /users/admin`, `/users/operario`, `/users/super-admin`.

---

## 3. Planes comerciales y módulos del menú

- Cada plan define qué **módulos** incluye.
- Una empresa puede tener **más módulos** que el plan base por contrato o **habilitación manual**: la lista efectiva = plan + extras; debe resolverse en servidor y reflejarse en login (ver §4).
- `SUPER_ADMIN`: sin filtro de módulos en UI.
- `ADMIN_EMPRESA`: menú filtrado por lista de módulos efectivos.

**Claves que usa el front** (`src/core/navigation/plan-modules.ts`):

`facturacion` · `ventas_pos` · `maestros` · `contabilidad` · `reportes_financieros` · `tesoreria` · `inventario` · `activos_fijos` · `rrhh` · `crm` · `estructura`

---

## 4. Login / sesión — `plan_modulos`

**Cambio en backend:** en la respuesta de `POST /auth/login`, incluir en `empresa_actual` el array `plan_modulos` (strings anteriores). Misma lógica por ítem en `mis_empresas` al cambiar de empresa.

| Condición | Comportamiento acordado con el front |
|-----------|--------------------------------------|
| `plan_modulos` ausente, `null` o `[]` | El front muestra todos los módulos |
| `plan_modulos` con valores | `ADMIN_EMPRESA` ve solo esas secciones; `SUPER_ADMIN` ignora el filtro |
| Habilitaciones manuales / upsell | Incluidas en la lista efectiva devuelta |

---

## 5. Configuración de empresa, logo, firma, avatar

| Área | Cambio / mejora en backend |
|------|----------------------------|
| `PATCH /companies/:razonSocialId/config` | DTO: agregar opcionales **`nombre_comercial`**, **`direccion_matriz`**; persistir en BD (p. ej. `razones_sociales`; sincronizar `tenants` si corresponde). |
| `PATCH /companies/:razonSocialId/logo` | Multipart campo `logo`; respuesta con URL o datos para refrescar UI. |
| Firma electrónica | `POST /electronic-signatures/upload-file`, `DELETE /electronic-signatures/current/:razonSocialId` alineados con payloads del front. |
| Perfil usuario | `PATCH /users/profile/complete`; login/JWT con datos necesarios para avatar y perfil. |

---

## 6. Empresas (tenant) — estado y baja

| Ruta | Método | Body / uso | Estado en front |
|------|--------|------------|-----------------|
| `/tenants/:tenantId/estado` | PATCH | `{ "estado": "activo" \| "inactivo" }` | Pantalla Empresas: toggle activo/inactivo |
| `/tenants/:tenantId` | DELETE | — | Pantalla Empresas: eliminar tras confirmación texto `ELIMINAR` |

**Implementar** ambas o documentar alternativa equivalente. Sin implementación → **404** en UI.

---

## 7. Usuarios — estado y baja

| Ruta | Método | Body / uso | Estado en front |
|------|--------|------------|-----------------|
| `/users/:userId` | PATCH | `{ "esta_activo": boolean }` (y otros campos editables si los exponéis) | Usuarios globales: activar/desactivar |
| `/users/:userId` | DELETE | — | Usuarios globales: eliminar tras confirmación |

**Implementar** PATCH para `esta_activo` como mínimo; DELETE según política de borrado lógico/físico. Sin rutas → **404**.

---

## 8. Suscripción del tenant

| Ruta | Método | Body |
|------|--------|------|
| `/tenants/:tenantId/subscription/plan` | PATCH | `{ "plan_id": "uuid" }` |
| `/tenants/:tenantId/subscription/vigencia` | PATCH | fechas inicio/fin; modalidad si aplica |

Alta/edición de empresa en front ya invoca estas rutas; deben existir o sustituirse por rutas equivalentes documentadas.

---

## 9. Respuestas de listados

**Cambio:** `GET /users` debe devolver **`es_admin_principal`** cuando exista en BD. Relaciones tenant / razón social completas para listados sin celdas vacías.

---

## 10. Cobertura actual Front vs trabajo Backend

| Módulo UI | Integración |
|-----------|-------------|
| Gestión global (empresas, usuarios, planes, dashboard, tipos operativo) | Parcial: faltan endpoints/DTOs de §5–§9; tipos operativo pueden depender de storage local hasta API dedicada. |
| Mi empresa | Listados usuarios/operarios; alinear permisos con §2. |
| Perfil (mi perfil, configuración empresa, contraseña) | Depende de §5 y sesión. |
| Resto del menú lateral (facturación, tesorería, inventario, RRHH, CRM, contabilidad, POS, etc.) | Mayoría **solo layout / título**; requiere APIs por dominio en fases posteriores. |

---

## 11. Checklist de implementación

- [ ] `POST /users/super-admin` → solo `email`, `password`, `razon_social_id`
- [ ] `PATCH /tenants/:id/estado`
- [ ] `DELETE /tenants/:id` (o equivalente con reglas definidas)
- [ ] `PATCH /users/:id` con `esta_activo`
- [ ] `DELETE /users/:id` (o política única de baja)
- [ ] `PATCH /tenants/:id/subscription/plan` y `.../vigencia`
- [ ] DTO `PATCH .../companies/.../config` → `nombre_comercial`, `direccion_matriz`
- [ ] Login → `empresa_actual.plan_modulos` (+ `mis_empresas` coherente)
- [ ] `GET /users` → `es_admin_principal` + relaciones completas
- [ ] Guards §2 en altas de usuario
