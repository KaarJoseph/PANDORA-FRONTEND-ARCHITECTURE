# 📌 Solicitud backend PANDORA

---

## 1) 🏢👤 Activar / desactivar empresa y usuario (prioridad)

Aquí hay **dos cosas distintas**. Si mezclas “eliminar” con “desactivar”, la app no se comporta bien.

### Empresa (tenant)

| Acción en la pantalla **Empresas** | Qué espera el front | Qué debe existir en backend |
|-----------------------------------|---------------------|-----------------------------|
| **Activar / Inactivar** (toggle) | `PATCH /tenants/:tenantId/estado` con body `{ "estado": "activo" \| "inactivo" }` | Implementar y persistir `estado` en el tenant. Sin esto la UI muestra error **404**. |
| **Eliminar** (modal, escribir ELIMINAR) | `DELETE /tenants/:tenantId` | Definir si es **borrado lógico** o físico y reglas (¿suscripciones?, ¿usuarios?). Sin esto **404**. |

👉 No es lo mismo **inactivar** (sigue en BD, no entra o queda bloqueado) que **eliminar** (baja del sistema según política).

### Usuario (personas del sistema)

| Acción en **Usuarios globales** | Qué espera el front | Qué debe existir en backend |
|--------------------------------|---------------------|-----------------------------|
| **Activar / Inactivar** (toggle menú) | `PATCH /users/:userId` con body `{ "esta_activo": true \| false }` (u otro campo acordado) | Hoy **no está** en el `UsersController` del Nest que revisamos: la UI recibe **404**. Es lo mínimo para “deshabilitar” sin borrar. |
| **Eliminar** (modal, escribir ELIMINAR) | `DELETE /users/:userId` | Opcional según negocio: borrado duro o marcar inactivo. Si no existe, **404**. El front lo usa como **borrado definitivo** tras confirmación. |

**Resumen humano:** primero cerrar **`PATCH /users/:id`** para `esta_activo` y **`PATCH /tenants/:id/estado`**; así dejan de fallar los toggles. **`DELETE`** va aparte (contrato claro: ¿se puede borrar un admin propietario?, etc.).

---

## 2) 💳 Suscripción: plan y vigencia (empresa)

El front ya llama esto al crear/editar empresa; si no existe, verás **404** y mensajes en pantalla.

| Ruta esperada | Body / uso |
|---------------|------------|
| `PATCH /tenants/:tenantId/subscription/plan` | `{ "plan_id": "uuid" }` |
| `PATCH /tenants/:tenantId/subscription/vigencia` | fechas inicio/fin (+ modalidad si aplica) |

Backend debe **crear** estos endpoints o **alias** documentados para que el front no tenga que adivinar.

---

## 3) 🏭 Configuración de empresa + logo + firma (perfil)

**Pantalla:** `Perfil → Configuración de empresa` (wizard: empresa, firma, facturación).

| Tema | Qué hacer en backend |
|------|----------------------|
| **DTO** `PATCH /companies/:razonSocialId/config` | Añadir al validador y persistir **`nombre_comercial`** y **`direccion_matriz`** (hoy el front los envía pero suelen **perderse** si el DTO no los acepta). Resto de campos fiscales según ya tengáis. |
| **Logo** | `PATCH /companies/:razonSocialId/logo` (multipart `logo`). Que devuelva URL o confirme guardado para refrescar sesión/listados. |
| **Firma electrónica** | `POST /electronic-signatures/upload-file`, `DELETE /electronic-signatures/current/:razonSocialId` — alinear con lo que ya usa el front. |

**Avatar del usuario** va por **`PATCH /users/profile/complete`** y lo que devuelva login/JWT para mostrar nombre/foto si un día la hay.

---

## 4) 🔐 Login y menú por plan

| Qué | Detalle |
|-----|---------|
| **`plan_modulos`** | Incluir en `empresa_actual` (y coherente en `mis_empresas` al cambiar de empresa) un array de strings: `facturacion`, `ventas_pos`, `maestros`, `contabilidad`, `reportes_financieros`, `tesoreria`, `inventario`, `activos_fijos`, `rrhh`, `crm`, `estructura`. |
| **Reglas** | Si no mandáis lista o va vacía → el front muestra **todo** (transición). Si mandáis lista → el **admin de empresa** solo ve esos bloques; **super admin** ignora el filtro. |
| **Extra manual** | Si un cliente paga más módulos o soporte habilita algo a mano, eso debe **verse** en la misma lista efectiva. |

Referencias en front: `src/core/navigation/plan-modules.ts`, tipos en `auth.types.ts`.

---

## 5) ⭐ Crear Super Admin (DTO mínimo)

**`POST /users/super-admin`** solo debe requerir:

- `email`
- `password` (hash en servidor)
- `razon_social_id` (vínculo a la razón social; en doc de producto a veces se nombra `razonSocialId`)

Quitar obligatoriedad de nombre/apellido en el **alta** si el flujo es completar perfil después.

---

## 6) 👥 Jerarquía: quién crea a quién

- **Super admin:** otros super admins (marco / empresa propietaria), admins para **cualquier** empresa cliente, y **operativos solo de la empresa propietaria** (la vuestra).
- **Admin de empresa:** solo **admins y operativos de su tenant** (su empresa).

Los `POST` ya existen en espíritu (`/users/admin`, `/users/operario`, `/users/super-admin`); backend debe **reforzar** con guards lo que el front no puede garantizar.

---

## 7) 📋 Listados: campos que deben venir bien

- **`es_admin_principal`** en `GET /users` (y donde aplique) para no depender de heurísticas en el front.
- Relaciones **tenant / razón social** consistentes para que no salga empresa en blanco.

---

## 8) 🗺️ Mapa: pantallas con lógica vs solo maquetas

| Zona | Estado |
|------|--------|
| **Gestión global** (dashboard, empresas, usuarios, planes, tipos operativo UI) | Hay pantallas y llamadas; faltan sobre todo **PATCH/DELETE tenant**, **PATCH/DELETE user**, **suscripción**, y DTO de **config** completo. **Tipos operativo** en global puede estar apoyado en storage local: valorar **API** propia. |
| **Mi empresa** (usuarios, tipos operativo) | Listados vía API de usuarios/operarios según rol; revisar permisos y **activar/inactivar** si lo mostráis en UI más adelante. |
| **Perfil** (mi perfil, configuración empresa, seguridad contraseña) | Flujo perfil + empresa + firma; pendiente persistencia de **nombre comercial / dirección matriz** en servidor. |
| **Todo el menú lateral** (facturación, tesorería, inventario, RRHH, CRM, contabilidad, POS, etc.) | En su mayoría son **páginas título solamente**: no hay módulos de negocio conectados. Cuando backend tenga APIs por dominio, el front las irá enganchando. **No** está en este doc enumerar 50 CRUDs: es trabajo por módulo cuando defináis prioridad. |

---

## 9) ✅ Checklist corto

- [ ] `PATCH /tenants/:id/estado` — activar/inactivar empresa  
- [ ] `DELETE /tenants/:id` o política clara de baja  
- [ ] `PATCH /users/:id` — al menos `esta_activo`  
- [ ] `DELETE /users/:id` — si aplica  
- [ ] `PATCH .../subscription/plan` y `.../vigencia`  
- [ ] DTO config: `nombre_comercial`, `direccion_matriz`  
- [ ] Login: `plan_modulos`  
- [ ] `GET /users` con `es_admin_principal` fiable  

---

*Si algo de esta lista ya está implementado en una rama, tachadlo en vuestro README interno; el front sigue estos paths hasta que cambiéis contrato de forma explícita.*
