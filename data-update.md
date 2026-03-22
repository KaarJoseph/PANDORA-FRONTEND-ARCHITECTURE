# 🚀 Backend PANDORA — Cambios y Mejoras Técnicas

Documento de alineación para la gestión de jerarquías, multi-tenancy y configuración de módulos.

---

## 1. 🔑 Arquitectura de Base de Datos y Lógica de Super Admin

Refactorización para la gestión de usuarios de alto nivel y **DTO de inicialización**.

### 1.1 `createSuperAdmin` — Parámetros de entrada
Se simplifica el alta: solo identidad y vínculo empresarial primario.

| Parámetro | Obligatorio | Notas |
|-----------|:-----------:|-------|
| `email` | ✅ Sí | Identificador único de acceso. |
| `password` | ✅ Sí | Persistir únicamente el Hash. |
| `razon_social_id` | ✅ Sí | Vinculación a la entidad legal (Razón Social). |

> [!IMPORTANT]
> **Cambio en Backend:** El endpoint `POST /users/super-admin` solo debe procesar estos 3 campos. Los datos personales (nombre/apellido) se gestionarán mediante `PATCH /users/profile/complete`.

---

## 2. 👥 Jerarquía de Creación de Usuarios

Lógica de permisos para la creación de cuentas según el rol del solicitante:

| Actor | Puede crear | Restricción / Scope |
|-------|-------------|---------------------|
| **SUPER_ADMIN** | Otros `SUPER_ADMIN` | Según reglas del Grupo Propietario. |
| **SUPER_ADMIN** | `ADMIN_EMPRESA` | Cualquier empresa (propias y clientes). |
| **SUPER_ADMIN** | `OPERATIVO` | Solo en el tenant de la **empresa propietaria**. |
| **ADMIN_EMPRESA**| `ADMIN_EMPRESA` y `OPERATIVO` | Solo usuarios de **su propio** tenant. |

---

## 3. 📦 Planes Comerciales y Módulos Aditivos

El sistema debe permitir una configuración flexible de "ventanas" o módulos visibles.

* **Lógica Aditiva:** Los módulos visibles para una empresa son el resultado de su **Plan Base + Habilitaciones Manuales (Extras)**.
* **Habilitación por Super Admin:** El `SUPER_ADMIN` tiene la potestad de activar módulos adicionales (upsell/ventanas extra) a cualquier empresa sin necesidad de cambiar su plan base.
* **Filtros en UI:**
    * `SUPER_ADMIN`: Visualiza todos los módulos (sin restricciones).
    * `ADMIN_EMPRESA`: Menú filtrado por la lista de módulos efectivos resuelta por el servidor.

**Claves de módulos (Slug):**
`facturacion` · `ventas_pos` · `maestros` · `contabilidad` · `reportes_financieros` · `tesoreria` · `inventario` · `activos_fijos` · `rrhh` · `crm` · `estructura`

---

## 4. 🔐 Login / Sesión — `plan_modulos`

**Cambio en backend:** El objeto de respuesta en `POST /auth/login` debe incluir en `empresa_actual` el array `plan_modulos` con los strings de los módulos habilitados.

| Condición | Comportamiento en Front-end |
|-----------|-----------------------------|
| Array ausente, `null` o `[]` | Se muestran todos los módulos por defecto. |
| Array con valores | El usuario ve **solo** los módulos listados. |
| Super Admin | Ignora el filtro y ve todo. |

---

## 5. 🏢 Configuración de Empresa, Logo y Firma

| Área | Cambio / Mejora Requerida |
|------|---------------------------|
| **Configuración** | `PATCH /companies/:razonSocialId/config`: Agregar `nombre_comercial` y `direccion_matriz`. |
| **Branding** | `PATCH /companies/:razonSocialId/logo`: Gestión de Multipart para el logo de la empresa. |
| **Firma Electrónica**| `POST /electronic-signatures/upload-file`: Vinculación de archivos `.p12` o certificados. |
| **Perfil** | `PATCH /users/profile/complete`: Para que el usuario final complete sus datos tras el alta. |

---

## 6. 🏢 Gestión de Tenants (Empresas) — Estado y Baja

Endpoints críticos para la administración de empresas clientes.

| Ruta | Método | Acción | Notas |
|------|:------:|--------|-------|
| `/tenants/:id/estado` | `PATCH` | Activar / Inactivar | Bloquea el acceso a todos los usuarios de ese tenant. |
| `/tenants/:id` | `DELETE`| Eliminar | Se recomienda **Borrado Lógico** (Soft Delete) para trazabilidad legal. |

---

## 7. 👤 Gestión de Usuarios — Estado y Baja

| Ruta | Método | Acción | Notas |
|------|:------:|--------|-------|
| `/users/:id` | `PATCH` | Cambiar `esta_activo` | Permite suspender usuarios individualmente sin borrarlos. |
| `/users/:id` | `DELETE`| Eliminar usuario | Validar que no sea el único Admin de una empresa activa. |

---

## 8. 📅 Suscripción del Tenant

Configuración de la vigencia y el nivel de servicio.

* `PATCH /tenants/:id/subscription/plan`: Cambia el `plan_id` (UUID).
* `PATCH /tenants/:id/subscription/vigencia`: Actualiza fechas de inicio y fin de contrato.

---

## 9. 📊 Listados y Reportes

* **`GET /users`**: Debe retornar obligatoriamente el booleano `es_admin_principal` y el objeto de relación con la Razón Social completo (evitar celdas vacías en la tabla).

---

## 10. ✅ Checklist de Implementación Final

- [ ] **Auth:** Inyectar `plan_modulos` en el payload de login (calculando Plan + Extras).
- [ ] **Empresas:** Implementar toggle de estado (activo/inactivo) y borrado lógico.
- [ ] **Módulos:** Endpoint para que el Super Admin habilite "ventanas" (módulos) manuales por empresa.
- [ ] **Usuarios:** Implementar DTO reducido para `createSuperAdmin`.
- [ ] **Config:** Soportar `nombre_comercial` y `direccion_matriz` en la tabla de razones sociales.
- [ ] **Seguridad:** Aplicar Guards de jerarquía según la tabla de la Sección 2.
