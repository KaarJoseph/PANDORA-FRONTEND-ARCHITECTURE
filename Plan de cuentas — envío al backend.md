# Plan de cuentas — envío al backend

## Qué hace el usuario y cuándo se sincroniza

- **Modo edición:** el usuario entra con **Editar**; ahí puede cambiar celdas, insertar filas (botones **+**) y eliminar filas. Nada de eso llega al servidor hasta **Guardar**.
- **Cancelar:** descarta los cambios locales y vuelve al último estado cargado desde el servidor (no se envía el JSON de sincronización).
- **Guardar:** el front arma un **snapshot**: todas las filas visibles de la **empresa activa** (`razon_social_id`) y las manda en un solo `POST` (sincronización masiva). Eso es lo que recibe el backend como estado deseado del plan.
- **Plan base / importación en pantalla:** solo llenan la tabla en el navegador; la persistencia ocurre **igual** al pulsar **Guardar** (mismo contrato JSON).
- **Regla de consistencia:** el listado que se envía define el plan completo para esa empresa: actualizar con `id`, crear sin `id`, y dar de baja lo que antes existía y ya no aparece en `cuentas[]` (ver tabla abajo).

## Punto HTTP

`POST /plan-cuentas/{razonSocialId}/bulk-sync`

`{razonSocialId}` en la URL debe ser **igual** a `razon_social_id` en el JSON.

## JSON que arma el frontend al guardar

```json
{
  "razon_social_id": "3f5d3f76-7d98-4c58-8ec6-a18a3f50b6ac",
  "cuentas": [
    {
      "id": "0b1576b4-5763-4ba2-b559-54a4d765e2ad",
      "codigo_cuenta": "1.01.01",
      "nombre_cuenta": "Caja General",
      "modulo": false,
      "permite_movimiento": true,
      "f101": "101",
      "patrimonio": "N",
      "super": "1.01",
      "flujo": "OPERATIVO"
    },
    {
      "codigo_cuenta": "1.01.02",
      "nombre_cuenta": "Caja Chica",
      "modulo": false,
      "permite_movimiento": true
    }
  ]
}
```

## Cómo debe aceptarlo el backend (resumo)

| En el payload | Significado |
|----------------|------------|
| Fila **con** `id` | Actualizar esa cuenta (debe ser de esa empresa). |
| Fila **sin** `id` | Crear cuenta nueva bajo `razon_social_id`. |
| Cuenta que **ya existía** en BD para esa empresa y **no** viene en `cuentas[]` | Tratarla como **eliminada** (sync por snapshot). |

**Campos por fila (tipos):** `modulo` y `permite_movimiento` son booleanos (`true` / `false`). El resto de strings opcionales solo se envían si tienen valor; `codigo_cuenta` y `nombre_cuenta` van siempre.
