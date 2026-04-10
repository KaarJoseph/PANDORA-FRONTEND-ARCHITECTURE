# Incidencia: error al guardar el plan de cuentas (`check_plan_deleted_coherence`)

## Objeto del documento

Se describe, con tono técnico y sin informalismos, en qué circunstancias de uso de la pantalla **Plan de cuentas** puede producirse un fallo al **guardar**, cuál es la manifestación en cliente y en servidor, y por qué distintas acciones del usuario convergen en el mismo tipo de error.

## Manifestación del fallo

- En el navegador, la petición a `POST …/plan-cuentas/import-json` concluye con **código HTTP 500**.
- En el registro del backend (NestJS con Prisma y PostgreSQL) aparece un error de integridad del tipo: la nueva fila (o la operación sobre `plan_cuentas`) **viola la restricción de comprobación** denominada `check_plan_deleted_coherence`.

Ese mensaje indica que la base de datos rechaza el estado resultante de la transacción, no que el cuerpo de la petición carezca de sintaxis JSON válida.

## Cuándo ocurre en la aplicación (contexto funcional)

### Primer supuesto: sustitución del plan tras incorporar la plantilla base

Cuando la empresa **ya dispone** de un plan de cuentas persistido y, desde la interfaz, se incorpora el **plan base** (plantilla) y a continuación se ejecuta **Guardar**, el cliente envía al servidor una **sustitución masiva** del plan mediante el endpoint `import-json`. En ese escenario se ha observado el error anterior.

### Segundo supuesto: modo edición y eliminación de cuentas

Cuando el usuario activa **Editar**, **elimina** una o varias líneas del plan (la aplicación puede retirar también las subcuentas asociadas según la acción de borrado confirmada) y pulsa **Guardar**, el conjunto de cuentas deja de coincidir con el que había en servidor al abrir la edición. La lógica del cliente interpreta entonces que debe aplicarse de nuevo una **sustitución completa** del plan, no únicamente parches puntuales, y vuelve a invocar el mismo `import-json`. En ese segundo supuesto **puede reproducirse el mismo error** que en el primero, puesto que la operación de persistencia es análoga desde el punto de vista del contrato HTTP.

## Interpretación (alcance y límites)

La restricción `check_plan_deleted_coherence` está definida a nivel de tabla en PostgreSQL; su texto exacto determina qué combinaciones de filas activas, referencias entre cuentas y posibles marcas de baja son admisibles. **No basta inferir** desde la interfaz si el conflicto proviene únicamente de “eliminar un padre” o de otro detalle jerárquico: la causa debe confirmarse revisando la definición SQL del *check* y el orden de operaciones dentro de la transacción del servicio que procesa `import-json`.

En suma: el fallo se asocia a **guardados que implican reemplazo masivo del plan** (tras mezclar plantilla con plan existente o tras eliminar filas en edición), y se manifiesta como rechazo de la base de datos bajo dicha restricción.

## Anexo (solo para reproducir técnicamente)

A efectos de pruebas aisladas (por ejemplo, con cliente HTTP), existe en el repositorio un cuerpo de ejemplo coherente con el formato que envía el front cuando las filas no incluyen catálogos opcionales: `plan-cuentas-import-json-payload-completo.json`. El identificador `razon_social_id` contenido en ese archivo debe sustituirse por el de la empresa objeto de la prueba.
