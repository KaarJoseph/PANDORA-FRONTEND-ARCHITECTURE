# 📋 Solicitud unificada Backend — PANDORA (referencia Frontend)

> **Un solo documento** para GitHub: todo lo pendiente o esperado desde el **frontend**, organizado por flujo.  
> Código de referencia: `src/modules/**/services`, `src/core/config/axios.ts`, `src/core/navigation/plan-modules.ts`.

---

## 📑 Índice

1. [Actualizaciones de arquitectura y lógica de Super Admin](#-actualizaciones-de-arquitectura-de-base-de-datos-y-lógica-de-super-admin) *(texto base del producto)*
2. [Jerarquía: quién crea a quién](#-jerarquía-de-usuarios-quién-crea-a-quién)
3. [Planes comerciales y módulos del menú](#-planes-comerciales-y-módulos-del-menú)
4. [Configuración de empresa, avatar y logo](#-configuración-de-empresa-avatar-y-logo)
5. [Sesión y `plan_modulos`](#-sesión--login-y-plan_modulos)
6. [Catálogo de endpoints consumidos por el front](#-catálogo-de-endpoints-consumidos-por-el-front)
7. [RBAC vs módulos de plan](#-rbac-fino-vs-módulos-de-plan)
8. [Resumen ejecutivo para backlog](#-resumen-ejecutivo-para-backlog)

---

## 🏛️ Actualizaciones de Arquitectura de Base de Datos y Lógica de Super Admin

Este documento detalla los cambios necesarios para la gestión de usuarios de alto nivel y la creación de la entidad **Super Admin**, enfocándose en la **jerarquía de privilegios** y la **simplificación de parámetros de inicialización**.

### 1️⃣ Actualización de parámetros: `createSuperAdmin`

Se debe refactorizar el método de creación de Super Admins para que sea **más conciso**. La estructura de datos de entrada (DTO) ahora se limitará estrictamente a la **identidad** y la **vinculación empresarial básica**.

#### ✅ Nuevos parámetros requeridos

| Campo | Descripción |
|--------|-------------|
| **email** | Identificador único de acceso. |
| **password** | Credencial de autenticación *(debe ser **hasheada** antes de la persistencia)*. |
| **razonSocialId** | Vinculación obligatoria con la entidad legal / empresa correspondiente *(en API REST suele exponerse como `razon_social_id`)*. |

> 🔧 **Nota técnica:** el frontend envía `email`, `password`, `razon_social_id` en `POST /users/super-admin`. No envía nombre/apellido en el alta; el perfil puede completarse después con `PATCH /users/profile/complete` o flujo equivalente.

---

## 👥 Jerarquía de usuarios: quién crea a quién

Estas reglas deben reflejarse en **permisos de API** y, si aplica, en **constraints** de base de datos.

| Rol | Qué puede crear | Alcance |
|-----|------------------|---------|
| 🌐 **SUPER_ADMIN** | Otros **Super Administradores** | Solo dentro del marco acordado (p. ej. vinculados a la **empresa propietaria** / razón social del grupo). |
| 🌐 **SUPER_ADMIN** | **Administradores de empresa** | Tanto para **su propia organización** como para **otras empresas (clientes)** que gestiona el grupo: son los tenants que comercialmente dependen de él. |
| 🏢 **ADMIN_EMPRESA** | **Operativos** | **Solo** operativos de **su** empresa (su **tenant** actual). |
| 🏢 **ADMIN_EMPRESA** | **Otros administradores** | **Solo** administradores de **su** empresa (mismo tenant), no de otras compañías. |

> 💡 **Idea clave:** el Super Admin “atiende” al ecosistema completo (propios + clientes); el administrador de una empresa cliente solo administra **su** compañía (usuarios internos: admins y operativos).

---

## 📦 Planes comerciales y módulos del menú

- Cada **plan** debe definir (o poder asociar) un conjunto de **módulos** habilitados.
- Una empresa puede **contratar más módulos** o recibir **habilitación manual** (fuera del catálogo estándar): eso debe **fusionarse** en la lista efectiva que ve el usuario (suscripción + overrides contractuales).
- El **Super Admin** **no** se limita por plan en la UI: ve **todos** los módulos de navegación.
- El **Administrador de empresa** ve solo los módulos incluidos en su **plan efectivo** (ver `plan_modulos` más abajo).

**Claves canónicas** alineadas con `src/core/navigation/plan-modules.ts`:

| Clave | Contenido aproximado del menú |
|--------|-------------------------------|
| `facturacion` | Facturación electrónica (facturas, notas, retenciones, guías, etc.) |
| `ventas_pos` | Punto de venta, mis ventas, cobros diarios, cierre de caja (`/ventas-pos/*`) |
| `maestros` | Personas, productos, categorías |
| `contabilidad` | Plan de cuentas, asientos, centros de costo, proyectos, periodos |
| `reportes_financieros` | Balance, estado de resultados, libro mayor, flujo de efectivo |
| `tesoreria` | Bancos, cuentas, cajas, CxC, CxP, cobros, pagos, conciliaciones |
| `inventario` | Bodegas, kardex, movimientos |
| `activos_fijos` | Activos y depreciaciones |
| `rrhh` | Empleados, nóminas, asistencias, préstamos, vacaciones |
| `crm` | Cotizaciones, proformas |
| `estructura` | Establecimientos, puntos de emisión, secuenciales, rangos autorizados |

---

## 🏢 Configuración de empresa, avatar y logo

### 📝 Campos de configuración (`PATCH /companies/:razonSocialId/config`)

La pantalla de **configuración de empresa** en el perfil permite capturar, entre otros, **nombre comercial** y **dirección matriz**.

- ⚠️ Hoy el **DTO del backend** puede **no incluir** `nombre_comercial` ni `direccion_matriz` en el validador, por lo que **se descartan** al guardar.
- ✅ **Pendiente backend:** añadir como **opcionales** `nombre_comercial` y `direccion_matriz` al DTO y persistirlos en **`razones_sociales`** (y, si aplica, sincronizar `tenants.nombre_comercial`).
- **Hasta entonces:** el front puede mantener un **snapshot local** (`localStorage`) tras guardar el resto de campos, pero la **fuente de verdad** debe ser el servidor.

### 🖼️ Avatar y logo

- El **perfil de usuario** (avatar, datos personales) se alinea con **`PATCH /users/profile/complete`** y la sesión extendida en NextAuth.
- El **logo de la empresa** usa **`PATCH /companies/:razonSocialId/logo`** (multipart).
- Cualquier **URL de avatar/logo** devuelta en login o GET de usuario debe ser **estable** y respetar **CORS** si se sirve desde otro dominio.

---

## 🔐 Sesión / login y `plan_modulos`

Para que el **administrador de empresa** vea solo las **ventanas contratadas**, el backend debe exponer los **módulos** del tenant actual.

### Ejemplo sugerido dentro de `POST /auth/login`

```json
"empresa_actual": {
  "tenant_id": "...",
  "tenant_slug": "...",
  "nombre": "...",
  "razon_social_id": "...",
  "logo_url": null,
  "plan_modulos": [
    "facturacion",
    "ventas_pos",
    "maestros",
    "contabilidad",
    "reportes_financieros",
    "tesoreria",
    "inventario",
    "activos_fijos",
    "rrhh",
    "crm",
    "estructura"
  ]
}
```

### Reglas acordadas con el frontend

| Situación | Comportamiento en UI |
|-----------|----------------------|
| `plan_modulos` **no viene**, es `null` o **array vacío** | Se muestran **todos** los módulos (modo transición / compatibilidad). |
| `plan_modulos` con **valores** | Solo se muestran secciones que coincidan *(solo aplica a `ADMIN_EMPRESA`; `SUPER_ADMIN` **ignora** el filtro)*. |
| **Overrides** contractuales (más módulos que el plan base) | Deben **mezclarse** en la lista efectiva que devuelve login o el servicio de suscripción. |

📎 Tipos: `LoginResponse` en `src/modules/auth/types/auth.types.ts` (`empresa_actual.plan_modulos?`, `mis_empresas[].plan_modulos?`).

---

## 📡 Catálogo de endpoints consumidos por el front

Base URL: la configurada en `NEXT_PUBLIC_API_URL` / `axios`.

### 🔑 Autenticación

| Método | Ruta |
|--------|------|
| POST | `/auth/login` |
| POST | `/auth/forgot-password` |
| POST | `/auth/reset-password` |

### 👤 Usuarios y perfiles

| Método | Ruta |
|--------|------|
| GET | `/users` |
| POST | `/users/super-admin` |
| POST | `/users/admin` |
| POST | `/users/operario` |
| GET | `/users/admin` |
| GET | `/users/operarios` |
| PATCH | `/users/profile/complete` |
| PATCH | `/users/:id` |
| DELETE | `/users/:id` |

### 🏭 Empresas, tenants y suscripción

| Método | Ruta |
|--------|------|
| GET | `/companies` |
| POST | `/companies` |
| PATCH | `/companies/:razonSocialId/config` |
| PATCH | `/companies/:razonSocialId/logo` |
| PATCH | `/tenants/:tenantId/estado` |
| DELETE | `/tenants/:tenantId` |
| PATCH | `/tenants/:tenantId/subscription/plan` |
| PATCH | `/tenants/:tenantId/subscription/vigencia` |

### 💳 Planes

| Método | Ruta |
|--------|------|
| GET | `/plans` |
| GET | `/plans/:id` |
| POST | `/plans` |
| PATCH | `/plans/:id` |

### 🏷️ Tipos operativo

| Método | Ruta |
|--------|------|
| GET | `/tipos-operativos` |

### ✍️ Firma electrónica

| Método | Ruta |
|--------|------|
| POST | `/electronic-signatures/upload-file` |
| DELETE | `/electronic-signatures/current/:razonSocialId` |

---

## ⚖️ RBAC fino vs módulos de plan

- **`plan_modulos`:** decide **qué áreas** del menú ve el **ADMIN_EMPRESA** (ventanas).
- **RBAC / permisos** (p. ej. `rol.permisos`): pueden restringir **acciones dentro de cada pantalla**; el front puede usarlos cuando el backend los exponga de forma estable.
- Mientras RBAC no esté unificado en todas las pantallas, el menú lateral se basa en **rol + `plan_modulos`** para administradores.

---

## ✅ Resumen ejecutivo para backlog

1. ✔️ **`POST /users/super-admin`:** DTO mínimo `email`, `password`, `razon_social_id` (hash en servidor).
2. ✔️ **`POST /auth/login`:** incluir `empresa_actual.plan_modulos` (y coherente al cambiar empresa en `mis_empresas`).
3. ✔️ **Planes:** mapear planes ↔ módulos; fusionar **suscripción + overrides manuales**.
4. ✔️ **`PATCH /companies/.../config`:** opcionales `nombre_comercial`, `direccion_matriz`.
5. ✔️ **Permisos de creación de usuarios** según jerarquía (Super Admin vs Admin de empresa).
6. ✔️ Validar todos los endpoints anteriores (existencia, errores, autorización por rol).

---

*Documento único sustituye a versiones anteriores fragmentadas; mantener este archivo como fuente de verdad para solicitudes al backend desde el frontend PANDORA.*
