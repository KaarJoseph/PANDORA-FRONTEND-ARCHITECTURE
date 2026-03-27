# ❌ Error al actualizar configuración de empresa (Prisma)

## 📌 Contexto

Se está intentando guardar información de configuración de una empresa en el backend (**PANDORA_BACKEND**), específicamente marcar a la empresa como **contribuyente especial**.

La operación se ejecuta desde el servicio:
- companies.service.ts
Usando Prisma dentro de una transacción: tx.establecimientos.updateMany()

---

## 👤 Escenario de prueba

Se realizaron pruebas con distintos tipos de usuarios:

- 🔹 **SuperAdmin Principal**
- 🔹 **SuperAdmin Normal**

### 🧪 Resultados

- **SuperAdmin Principal**
  - ✅ Puede intentar editar (comportamiento correcto)
  - ❌ Lanza error de Prisma (`column 'new' does not exist`)

- **SuperAdmin Normal**
  - ❌ No debería poder editar
  - ⚠️ Actualmente también llega a ejecutar lógica (posible falta de validación)
---

## ⚠️ Comportamiento esperado

- ✅ **SuperAdmin Principal**
  - Puede editar configuración sin errores

- ❌ **SuperAdmin Normal**
  - No debería poder editar
  - Debería recibir un error controlado (ej: `403 Forbidden`)

---

## 🚨 Error presentado

```bash
[Nest] ERROR [ExceptionsHandler] 
Invalid `tx.establecimientos.updateMany()` invocation

The column `new` does not exist in the current database.

```bash
[Nest] 42336  - 25/03/2026, 5:12:24 p. m.   ERROR [ExceptionsHandler] 
Invalid `tx.establecimientos.updateMany()` invocation in
C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\companies.service.ts:1417:47   

  1414 }
  1415
  1416 if (Object.keys(estData).length > 0) {
→ 1417     await tx.establecimientos.updateMany(
The column `new` does not exist in the current database.
PrismaClientKnownRequestError: 
Invalid `tx.establecimientos.updateMany()` invocation in
C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\companies.service.ts:1417:47   

  1414 }
  1415
  1416 if (Object.keys(estData).length > 0) {
→ 1417     await tx.establecimientos.updateMany(
The column `new` does not exist in the current database.
    at zr.handleRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:237:13)
    at zr.handleAndLogRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:183:12)
    at zr.request (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:152:12)
    at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
    at async a (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\getPrismaClient.ts:808:24)
    at async <anonymous> (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\companies.service.ts:1417:21)
    at async Proxy._transactionWithCallback (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\getPrismaClient.ts:697:18)
    at async CompaniesService.updateCompanyConfiguration (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\companies.service.ts:1266:16)
    at async C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@nestjs\core\router\router-execution-context.js:46:28
    at async C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@nestjs\core\router\router-proxy.js:9:17
```



## 👤 Escenario de prueba (Caso 2)

- 🔹 **Administrador de otra empresa (multi-tenant)**

---

### 🧪 Resultados

- **Administrador de otra empresa**
  - ❌ Puede intentar editar configuración de una empresa que no le pertenece  
  - ❌ Se ejecuta lógica en backend (no debería)  
  - ❌ Lanza error de Prisma (`column 'new' does not exist`)  

---

### ⚠️ Comportamiento esperado (Caso 2)

- ❌ **Administrador de otra empresa**
  - No debería poder editar configuraciones de empresas externas  
  - No debería ejecutar lógica de actualización  
  - Debería recibir un error controlado (ej: `403 Forbidden`)

 ```bash
[Nest] 44472  - 25/03/2026, 10:16:16 p. m.   ERROR [ExceptionsHandler] 
Invalid `tx.establecimientos.updateMany()` invocation in
C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\companies.service.ts:1417:47

  1414 }
  1415 
  1416 if (Object.keys(estData).length > 0) {
→ 1417     await tx.establecimientos.updateMany(
The column `new` does not exist in the current database.
PrismaClientKnownRequestError: 
Invalid `tx.establecimientos.updateMany()` invocation in
C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\companies.service.ts:1417:47

  1414 }
  1415 
  1416 if (Object.keys(estData).length > 0) {
→ 1417     await tx.establecimientos.updateMany(
The column `new` does not exist in the current database.
    at zr.handleRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:237:13)
    at zr.handleAndLogRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:183:12)
    at zr.request (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:152:12)
    at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
    at async a (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\getPrismaClient.ts:808:24)
    at async <anonymous> (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\companies.service.ts:1417:21)
    at async Proxy._transactionWithCallback (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\getPrismaClient.ts:697:18)
    at async CompaniesService.updateCompanyConfiguration (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\companies.service.ts:1266:16)
    at async C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@nestjs\core\router\router-execution-context.js:46:28
    at async C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@nestjs\core\router\router-proxy.js:9:17

```

---

```TEXT
ERROR LINK: tenantService.ts:35  PATCH http://localhost:3000/api/v1/companies/cd9ca234-a72a-4f63-95da-bef93483d592/config 500 (Internal Server Error)
```

## 👤 Escenario de prueba (Caso 3)

- 🔹 **Gestión de Permisos Dinámicos (`permisos_overrides_json`)**

---

### 🧪 Resultados

- **SuperAdmin Principal**
  - ✅ Intenta habilitar módulos o "ventanas" adicionales (Permisos Overrides).
  - ❌ **Fallo de Persistencia:** Los cambios en las ventanas extras no se guardan de forma adecuada.
  - ❌ **Rebote de Transacción:** Al intentar registrar los permisos, la lógica se interrumpe.
  - ❌ **Error de Prisma:** Lanza `column 'new' does not exist` al ejecutar la actualización en `establecimientos`.

---

### ⚠️ Comportamiento esperado (Caso 3)

- ✅ **Persistencia de Overrides:** El sistema debe permitir guardar el objeto `permisos_overrides_json` para habilitar ventanas específicas sin alterar el plan base.
- ✅ **Sincronización Dinámica:** La empresa debe ver reflejada la nueva ventana o módulo habilitado inmediatamente tras el guardado.
- ✅ **Integridad de Datos:** La transacción de Prisma debe procesar el campo `JSONB` correctamente sin intentar mapear llaves internas como columnas de la tabla.

---

### 👤 Escenario de prueba (Caso 4)
🔹 Edición de datos de facturación con conflicto de unicidad

---

### 🧪 Resultados
- ❌ No se pueden editar campos como **establecimiento (codigo_sri)** si ya existe uno igual
- ❌ Backend lanza error de Prisma por restricción única
- ⚠️ El frontend muestra éxito parcial pero no refleja cambios
- ✅ El **logo de empresa sí se puede editar** (no afecta restricciones)

---

### 💥 Error detectado

```bash
[Nest] 11332  - 26/03/2026, 1:56:46 p. m.   ERROR [ExceptionsHandler] 
Invalid `tx.establecimientos.update()` invocation in
C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\company-configuration.service.ts:235:73

  232 }
  233 if (Object.keys(estData).length > 0) {
  234     if (canUpdateEstablecimientos) {
→ 235         establecimiento = await tx.establecimientos.update(
Unique constraint failed on the fields: (`razon_social_id`, `codigo_sri`)
PrismaClientKnownRequestError: 
Invalid `tx.establecimientos.update()` invocation in
C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\company-configuration.service.ts:235:73

  232 }
  233 if (Object.keys(estData).length > 0) {
  234     if (canUpdateEstablecimientos) {
→ 235         establecimiento = await tx.establecimientos.update(
Unique constraint failed on the fields: (`razon_social_id`, `codigo_sri`)
    at zr.handleRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:237:13)
    at zr.handleAndLogRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:183:12)
    at zr.request (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:152:12)
    at zr.handleAndLogRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:183:12)
    at zr.request (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runt    at zr.handleAndLogRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:183:12)
    at zr.handleAndLogRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prism    at zr.handleAndLogRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:183:12)
    at zr.request (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:152:12)
    at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
    at async a (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\getPrismaClient.ts:808:24)
    at async <anonymous> (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\company-configuration.service.ts:235:47)
    at zr.handleAndLogRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:183:12)
    at zr.request (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:152:12)
    at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
    at async a (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\getPrismaClient.ts:808:24)
    at async <anonymous> (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\service    at zr.handleAndLogRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:183:12)
    at zr.request (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:152:12)
    at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
    at async a (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime    at zr.handleAndLogRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:183:12)
    at zr.handleAndLogRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prism    at zr.handleAndLogRequestError (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:183:12)
    at zr.request (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\RequestHandler.ts:152:12)
    at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
    at async a (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\getPrismaClient.ts:808:24)
    at async <anonymous> (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\company-configuration.service.ts:235:47)
    at async Proxy._transactionWithCallback (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@prisma\client\src\runtime\getPrismaClient.ts:697:18)
    at async CompanyConfigurationService.updateCompanyConfiguration (C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\src\modules\companies\services\company-configuration.service.ts:132:16)
    at async C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@nestjs\core\router\router-execution-context.js:46:28
    at async C:\Users\Joseph\Documents\FSO\BACKEND\PANDORA_BACKEND\node_modules\@nestjs\core\router\router-proxy.js:9:17
```


