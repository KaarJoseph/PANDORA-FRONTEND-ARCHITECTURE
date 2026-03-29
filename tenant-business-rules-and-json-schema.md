# 📋 Empresas (Super Admin): JSON y reglas de negocio — alineación backend

💡 Este documento define las reglas de negocio y la estructura de los datos necesarios para la creación y edición de empresas desde el backend. El objetivo es asegurar consistencia en la lógica, validaciones y comportamiento del sistema.

---

## 📌 Reglas de negocio

### 🪟 Ventanas / permisos (por empresa)

Cada plan define un conjunto base de ventanas incluidas por defecto.

- No está permitido eliminar ventanas que formen parte del plan base.
- Se permite agregar ventanas adicionales únicamente si el plan lo contempla como ampliación.
- El campo `permisos_override_json` debe utilizarse exclusivamente para extender permisos, no para restringirlos.

➕ El override de permisos es estrictamente aditivo.

---

### 👥 Usuarios (administradores y operativos)

El plan establece un número base de usuarios.

- No se permite definir límites por debajo del mínimo establecido por el plan.
- Se permite incrementar la capacidad mediante usuarios adicionales.

La representación de los límites debe ser consistente tanto en creación como en edición:

**📊 base del plan + adicionales = total efectivo**

Esta relación debe mantenerse clara y uniforme en todas las operaciones.

---

### 💰 Precio

El precio base se obtiene del plan, pero debe ser configurable a nivel de empresa.

- Se debe permitir definir un precio personalizado para cada empresa.
- El valor puede diferir del precio estándar del plan según acuerdos comerciales.

---

## 🧩 Estructura de los JSON

## ✏️ Crear empresa — ajuste de payload

En el proceso de creación de empresa desde Super Admin, no se deben incluir datos que corresponden a la configuración interna del tenant. Información como dirección matriz u obligación contable será gestionada posteriormente por la propia empresa.

Por lo tanto, el payload de creación debe limitarse a datos administrativos, plan, capacidades y configuración inicial del servicio.

### 📦 Payload corregido

```json
{
  "tenant_nombre_comercial": "Mi Cliente",
  "plan_id": "uuid-del-plan",
  "fecha_activacion": "2026-03-01T00:00:00.000Z",
  "fecha_expiracion": "2027-03-01T00:00:00.000Z",
  "ruc": "1234567890001",
  "razon_social": "CLIENTE SA",
  "tipo_negocio": "MIXTO",
  "limite_admins_override": 5,
  "limite_operativos_override": 30,
  "permisos_override_json": {
    "modulos": {}
  },
  "precio_final": 120.0
}

```

Como alternativa, se puede trabajar con campos explícitos de adicionales:

- extras_admins
- extras_operativos

y calcular el total internamente.

## 🔄 Editar empresa — misma lógica que creación

Las operaciones de edición deben seguir exactamente los mismos criterios definidos para la creación. No se deben manejar estructuras separadas; el objetivo es trabajar con un único payload coherente que represente el estado completo de la empresa.

### 📦 Payload unificado de edición

```json
{
  "plan_id": "uuid",
  "fecha_activacion": "2026-03-01T00:00:00.000Z",
  "fecha_expiracion": "2027-03-01T00:00:00.000Z",
  "limite_admins_override": 5,
  "limite_operativos_override": 30,
  "permisos_override_json": {
    "modulos": {}
  },
  "precio_final": 120.0
}
```

## 🔐 Estructura de permisos — definición normalizada (perfil administrador)

Se propone estructurar los permisos en tres niveles: **módulo → submódulo → acciones**, permitiendo una definición más clara y ordenada del acceso dentro del sistema.

Este enfoque permite manejar los permisos de forma más granular y consistente, evitando ambigüedades y facilitando su implementación tanto en backend como en frontend.

### ✅ Criterios

- Se utilizan acciones estándar como `ver`, `crear`, `editar`, `anular`, `eliminar`.
- Las acciones específicas (`emitir`, `conciliar`, `calcular`, etc.) se aplican únicamente cuando representan un proceso distinto.
- Se diferencia claramente entre:
  - `crear`: generación interna del documento
  - `emitir`: proceso de envío o validación electrónica

---

## 🗂️ Estructura propuesta (perfil administrador / propietario)

```json
{
  "modulos": {
    "ventas_pos": {
      "punto_venta": ["ver", "abrir_caja", "cerrar_caja"],
      "mis_ventas": ["ver", "crear", "editar", "anular"],
      "cobros": ["ver", "registrar", "anular"],
      "cierre_caja": ["ver", "ejecutar"]
    },
    "facturacion": {
      "facturas": ["ver", "crear", "editar", "anular", "emitir"],
      "notas_credito": ["ver", "crear", "editar", "anular", "emitir"],
      "notas_debito": ["ver", "crear", "editar", "anular", "emitir"],
      "retenciones": ["ver", "crear", "emitir", "anular"],
      "guias_remision": ["ver", "crear", "emitir", "anular"]
    },
    "personas": {
      "personas": ["ver", "crear", "editar", "eliminar"],
      "productos_servicios": ["ver", "crear", "editar", "eliminar"],
      "categorias": ["ver", "crear", "editar", "eliminar"]
    },
    "contabilidad": {
      "plan_cuentas": ["ver", "crear", "editar"],
      "asientos_contables": ["ver", "crear", "editar", "anular"],
      "centros_costo": ["ver", "crear", "editar"],
      "proyectos": ["ver", "crear", "editar"],
      "periodos_contables": ["ver", "abrir", "cerrar"]
    },
    "reportes_financieros": {
      "balance_general": ["ver", "exportar"],
      "estado_resultados": ["ver", "exportar"],
      "libro_mayor": ["ver", "exportar"],
      "flujo_efectivo": ["ver", "exportar"]
    },
    "tesoreria": {
      "gestion_bancaria": ["ver", "crear", "editar"],
      "cajas": ["ver", "crear", "editar", "cerrar"],
      "cxc": ["ver", "gestionar"],
      "cxp": ["ver", "gestionar"],
      "cobros": ["ver", "registrar", "anular"],
      "pagos": ["ver", "registrar", "anular"],
      "conciliaciones": ["ver", "conciliar"]
    },
    "inventario": {
      "bodegas": ["ver", "crear", "editar"],
      "kardex": ["ver"],
      "movimientos": ["ver", "crear", "anular"]
    },
    "activos_fijos": {
      "activos_fijos": ["ver", "crear", "editar", "dar_baja"],
      "depreciaciones": ["ver", "calcular"]
    },
    "rrhh": {
      "empleados": ["ver", "crear", "editar"],
      "nominas": ["ver", "generar"],
      "asistencias": ["ver", "registrar"]
    },
    "crm_preventa": {
      "cotizaciones": ["ver", "crear", "editar", "anular"],
      "proformas": ["ver", "crear", "editar", "anular"]
    }
  }
}
```

## 🗂️ Estructura propuesta basado en ROLES.


```json

{
  "planes": {
    "FACTURACION": {
      "modulos": {
        "ventas_pos": {
          "punto_venta": ["ver", "abrir_caja", "cerrar_caja"],
          "mis_ventas": ["ver", "crear"],
          "cobros": ["ver", "registrar"],
          "cierre_caja": ["ver"]
        },
        "facturacion": {
          "facturas": ["ver", "crear", "emitir"],
          "notas_credito": ["ver", "crear", "emitir"],
          "notas_debito": ["ver", "crear", "emitir"],
          "retenciones": ["ver", "crear", "emitir"],
          "guias_remision": ["ver", "crear", "emitir"]
        },
        "personas": {
          "personas": ["ver", "crear", "editar"],
          "productos_servicios": ["ver", "crear", "editar"],
          "categorias": ["ver", "crear"]
        },
        "inventario": {
          "bodegas": ["ver"],
          "kardex": ["ver"],
          "movimientos": ["ver", "crear", "anular"]
        }
      }
    },
    "PANDORA": {
      "modulos": {
        "ventas_pos": {
          "punto_venta": ["ver", "abrir_caja", "cerrar_caja"],
          "mis_ventas": ["ver", "crear", "editar", "anular"],
          "cobros": ["ver", "registrar", "anular"],
          "cierre_caja": ["ver", "ejecutar"]
        },
        "facturacion": {
          "facturas": ["ver", "crear", "editar", "anular", "emitir"],
          "notas_credito": ["ver", "crear", "editar", "anular", "emitir"],
          "notas_debito": ["ver", "crear", "editar", "anular", "emitir"],
          "retenciones": ["ver", "crear", "emitir", "anular"],
          "guias_remision": ["ver", "crear", "emitir", "anular"]
        },
        "personas": {
          "personas": ["ver", "crear", "editar"],
          "productos_servicios": ["ver", "crear", "editar"],
          "categorias": ["ver", "crear", "editar"]
        },
        "contabilidad": {
          "plan_cuentas": ["ver"],
          "asientos_contables": ["ver"],
          "centros_costo": ["ver"],
          "proyectos": ["ver"],
          "periodos_contables": ["ver"]
        },
        "reportes_financieros": {
          "balance_general": ["ver"],
          "estado_resultados": ["ver"],
          "libro_mayor": ["ver"],
          "flujo_efectivo": ["ver"]
        },
        "tesoreria": {
          "gestion_bancaria": ["ver"],
          "cajas": ["ver"],
          "cxc": ["ver", "gestionar"],
          "cxp": ["ver", "gestionar"],
          "cobros": ["ver", "registrar", "anular"],
          "pagos": ["ver", "registrar", "anular"],
          "conciliaciones": ["ver", "conciliar"]
        },
        "inventario": {
          "bodegas": ["ver", "crear", "editar"],
          "kardex": ["ver"],
          "movimientos": ["ver", "crear", "anular"]
        },
        "activos_fijos": {
          "activos_fijos": ["ver"],
          "depreciaciones": ["ver"]
        },
        "rrhh": {
          "empleados": ["ver"],
          "nominas": ["ver"],
          "asistencias": ["ver"]
        },
        "crm_preventa": {
          "cotizaciones": ["ver", "crear"],
          "proformas": ["ver", "crear"]
        }
      }
    },
    "PANDORA_ERP": {
      "modulos": {
        "ventas_pos": {
          "punto_venta": ["ver", "abrir_caja", "cerrar_caja"],
          "mis_ventas": ["ver", "crear", "editar", "anular"],
          "cobros": ["ver", "registrar", "anular"],
          "cierre_caja": ["ver", "ejecutar"]
        },
        "facturacion": {
          "facturas": ["ver", "crear", "editar", "anular", "emitir"],
          "notas_credito": ["ver", "crear", "editar", "anular", "emitir"],
          "notas_debito": ["ver", "crear", "editar", "anular", "emitir"],
          "retenciones": ["ver", "crear", "emitir", "anular"],
          "guias_remision": ["ver", "crear", "emitir", "anular"]
        },
        "personas": {
          "personas": ["ver", "crear", "editar", "eliminar"],
          "productos_servicios": ["ver", "crear", "editar", "eliminar"],
          "categorias": ["ver", "crear", "editar", "eliminar"]
        },
        "contabilidad": {
          "plan_cuentas": ["ver", "crear", "editar"],
          "asientos_contables": ["ver", "crear", "editar", "anular"],
          "centros_costo": ["ver", "crear", "editar"],
          "proyectos": ["ver", "crear", "editar"],
          "periodos_contables": ["ver", "abrir", "cerrar"]
        },
        "reportes_financieros": {
          "balance_general": ["ver", "exportar"],
          "estado_resultados": ["ver", "exportar"],
          "libro_mayor": ["ver", "exportar"],
          "flujo_efectivo": ["ver", "exportar"]
        },
        "tesoreria": {
          "gestion_bancaria": ["ver", "crear", "editar"],
          "cajas": ["ver", "crear", "editar", "cerrar"],
          "cxc": ["ver", "gestionar"],
          "cxp": ["ver", "gestionar"],
          "cobros": ["ver", "registrar", "anular"],
          "pagos": ["ver", "registrar", "anular"],
          "conciliaciones": ["ver", "conciliar"]
        },
        "inventario": {
          "bodegas": ["ver", "crear", "editar"],
          "kardex": ["ver"],
          "movimientos": ["ver", "crear", "anular"]
        },
        "activos_fijos": {
          "activos_fijos": ["ver", "crear", "editar", "dar_baja"],
          "depreciaciones": ["ver", "calcular"]
        },
        "rrhh": {
          "empleados": ["ver", "crear", "editar"],
          "nominas": ["ver", "generar"],
          "asistencias": ["ver", "registrar"]
        },
        "crm_preventa": {
          "cotizaciones": ["ver", "crear", "editar", "anular"],
          "proformas": ["ver", "crear", "editar", "anular"]
        }
      }
    }
  }
}
```


