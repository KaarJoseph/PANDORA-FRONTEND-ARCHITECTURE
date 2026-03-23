# 🏗️ Arquitectura de Permisos y Planes - Pandora ERP

Este documento detalla la estructura de datos para el control de acceso, los límites de suscripción y la gestión de políticas para usuarios operativos y administradores.

---

## 1. Definición Global de Planes y Políticas
A continuación se presenta el esquema JSON que rige el comportamiento de los módulos y las capacidades de cada nivel de suscripción dentro del ecosistema de **Pandora**.

```json
{
  "planes": {
    "FACTURACION": {
      "limites": {
        "max_admins": 1,
        "max_operativos": 2,
        "max_total_users": 3
      },
      "modulos": {
        "facturacion": [
          "ver",
          "crear",
          "editar",
          "anular",
          "emitir_nota_credito",
          "emitir_nota_debito",
          "emitir_retencion",
          "emitir_guia_remision"
        ],
        "ventas_pos": [
          "ver",
          "abrir_caja",
          "crear_venta",
          "registrar_cobro",
          "cerrar_caja"
        ],
        "personas": [
          "ver",
          "crear",
          "editar"
        ],
        "contabilidad": [],
        "reportes_financieros": [],
        "tesoreria": [],
        "inventario": [
          "ver",
          "crear",
          "editar"
        ],
        "activos_fijos": [],
        "rrhh": [],
        "crm": []
      }
    },
    "PANDORA": {
      "limites": {
        "max_admins": 1,
        "max_operativos": 4,
        "max_total_users": 5
      },
      "modulos": {
        "facturacion": [
          "ver",
          "crear",
          "editar",
          "anular",
          "emitir_nota_credito",
          "emitir_nota_debito",
          "emitir_retencion",
          "emitir_guia_remision"
        ],
        "ventas_pos": [
          "ver",
          "abrir_caja",
          "crear_venta",
          "registrar_cobro",
          "cerrar_caja",
          "reimprimir_comprobante"
        ],
        "personas": [
          "ver",
          "crear",
          "editar",
          "eliminar"
        ],
        "contabilidad": [
          "ver"
        ],
        "reportes_financieros": [
          "ver"
        ],
        "tesoreria": [
          "ver",
          "registrar_cobro",
          "registrar_pago",
          "conciliar",
          "gestionar_bancos",
          "gestionar_cuentas",
          "gestionar_tarjetas"
        ],
        "inventario": [
          "ver",
          "crear",
          "editar",
          "eliminar",
          "ajustar_stock",
          "ver_kardex"
        ],
        "activos_fijos": [
          "ver"
        ],
        "rrhh": [
          "ver",
          "ver_asistencia",
          "ver_nomina"
        ],
        "crm": [
          "ver_cotizaciones",
          "crear_cotizacion",
          "editar_cotizacion",
          "ver_proformas",
          "crear_proforma",
          "editar_proforma",
          "gestionar_establecimientos"
        ]
      }
    },
    "PANDORA_ERP": {
      "limites": {
        "max_admins": 1,
        "max_operativos": 14,
        "max_total_users": 15
      },
      "modulos": {
        "facturacion": [
          "ver",
          "crear",
          "editar",
          "anular",
          "emitir_nota_credito",
          "emitir_nota_debito",
          "emitir_retencion",
          "emitir_guia_remision",
          "configurar_series"
        ],
        "ventas_pos": [
          "ver",
          "abrir_caja",
          "crear_venta",
          "registrar_cobro",
          "cerrar_caja",
          "reimprimir_comprobante",
          "gestionar_arqueos"
        ],
        "personas": [
          "ver",
          "crear",
          "editar",
          "eliminar",
          "importar",
          "exportar"
        ],
        "contabilidad": [
          "ver",
          "crear_asiento",
          "editar_asiento",
          "anular_asiento",
          "gestionar_plan_cuentas",
          "gestionar_centros_costo",
          "gestionar_periodos",
          "gestionar_proyectos"
        ],
        "reportes_financieros": [
          "ver",
          "exportar",
          "programar_envios"
        ],
        "tesoreria": [
          "ver",
          "crear_movimiento",
          "editar_movimiento",
          "anular_movimiento",
          "gestionar_bancos",
          "gestionar_cuentas",
          "gestionar_tarjetas",
          "gestionar_cajas",
          "conciliar"
        ],
        "inventario": [
          "ver",
          "crear",
          "editar",
          "eliminar",
          "ajustar_stock",
          "ver_kardex",
          "gestionar_bodegas",
          "gestionar_movimientos"
        ],
        "activos_fijos": [
          "ver",
          "crear",
          "editar",
          "dar_baja",
          "calcular_depreciacion"
        ],
        "rrhh": [
          "ver",
          "gestionar_empleados",
          "gestionar_nomina",
          "gestionar_asistencia"
        ],
        "crm": [
          "ver_cotizaciones",
          "crear_cotizacion",
          "editar_cotizacion",
          "aprobar_cotizacion",
          "ver_proformas",
          "crear_proforma",
          "editar_proforma",
          "aprobar_proforma",
          "gestionar_establecimientos"
        ]
      }
    }
  },
  "empresa_overrides": {
    "tenant_id": "tenant_x",
    "plan_base": "PANDORA",
    "extra_slots": {
      "admins": 0,
      "operativos": 3
    },
    "extra_modulos_habilitados": [
      "rrhh"
    ],
    "modulos_bloqueados": [],
    "effective_policy": {
      "max_admins": 1,
      "max_operativos": 7,
      "max_total_users": 8,
      "modulos_habilitados": [
        "facturacion",
        "ventas_pos",
        "personas",
        "contabilidad",
        "reportes_financieros",
        "tesoreria",
        "inventario",
        "activos_fijos",
        "rrhh",
        "crm"
      ]
    }
  },
  "operativo_policy_template": {
    "scope": "OPERATIVO",
    "restricciones": {
      "no_puede_superar_modulos_empresa": true,
      "no_puede_gestionar_usuarios": true,
      "no_puede_editar_plan_empresa": true
    },
    "asignacion_por_admin": {
      "modulos_habilitados": [],
      "permisos_por_modulo": {
        "facturacion": [],
        "ventas_pos": [],
        "personas": [],
        "contabilidad": [],
        "reportes_financieros": [],
        "tesoreria": [],
        "inventario": [],
        "activos_fijos": [],
        "rrhh": [],
        "crm": []
      }
    }
  }
}
```

## 2. Lógica de Aplicación

### 🏢 Gestión de Overrides (Empresa)
El sistema permite que un **tenant** herede de un `plan_base` pero aplique modificaciones específicas sin romper la estructura global:

* **Extra Slots:** Incrementa la capacidad de usuarios contratados (Admins/Operativos) por encima del límite estándar del plan.
* **Effective Policy:** Es el objeto de resultado final calculado. El **frontend** debe consultar esta política para renderizar componentes, menús y proteger rutas de navegación.

### 👥 Perfil Operativo
Los usuarios operativos tienen un alcance restringido definido por el `operativo_policy_template`:

* **Herencia Inviolable:** Los módulos y permisos asignados a un operativo **deben** ser obligatoriamente un subconjunto de los `modulos_habilitados` en la `effective_policy` de la empresa.
* **Restricciones de Admin:** Por diseño, los perfiles operativos tienen bloqueada cualquier gestión de usuarios, creación de nuevos accesos o modificaciones al plan de suscripción de la empresa.
