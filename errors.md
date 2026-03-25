# ❌ Error al actualizar configuración de empresa (Prisma)

## 📌 Contexto

Se está intentando guardar información de configuración de una empresa en el backend (**PANDORA_BACKEND**), específicamente marcar a la empresa como **contribuyente especial**.

Durante este proceso, se ejecuta una operación `updateMany` sobre la tabla `establecimientos` dentro de una transacción de Prisma.

---

## 🚨 Error presentado

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
