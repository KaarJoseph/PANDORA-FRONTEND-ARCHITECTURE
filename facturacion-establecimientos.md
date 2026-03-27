# 🧾 Guía Completa: Establecimientos y Facturación Electrónica (Ecuador - SRI)

---

## 🎯 Objetivo

Explicar de forma clara:

- Cómo funciona la estructura de facturación en Ecuador
- Qué significan los códigos (001-001-000000001)
- Qué debe hacer el sistema automáticamente
- Cómo diseñar el flujo correcto (UX + Backend)

---

## 🏢 1. Estructura general de facturación

En Ecuador, cada documento electrónico (factura, nota, etc.) tiene este formato:

**ESTABLECIMIENTO - PUNTO EMISIÓN - SECUENCIAL**

Ejemplo:
001-001-000000001

---

## 🔍 2. ¿Qué significa cada parte?

### 🏢 Establecimiento (001)

- Es el código del local o sucursal
- Siempre tiene **3 dígitos**
- Ejemplos:
  - `001` → MATRIZ (principal)
  - `002` → Sucursal
  - `003` → Otra sucursal

📌 REGLA:
> Toda empresa SIEMPRE tiene al menos un establecimiento: la MATRIZ (001). Pero hay casos en los que no siempre será así, por lo tanto, Pandora no debería "hardcodear" (dejar fijo) ese valor.

---

### 🧾 Punto de emisión (001)

- Es una subdivisión del establecimiento
- Representa desde dónde se emite el comprobante
- También tiene **3 dígitos**

Ejemplo:
- `001` → Caja principal
- `002` → Caja secundaria

📌 Ejemplo completo:

**001-002**

→ **Establecimiento matriz, punto de emisión 2**

---

### 🔢 Secuencial (000000001)

- Número de la factura
- Tiene **9 dígitos**
- Es incremental

Ejemplo:

000000001
000000002
000000003

---

## ❓ 3. ¿Siempre empieza en 001-001?

### ✅ RESPUESTA CLARA:

### ✔ SÍ (por defecto en el sistema)

Cuando una empresa es nueva:

- Establecimiento → `001` (MATRIZ)
- Punto de emisión → `001`
- Secuencial → `000000001`

Resultado:

001-001-000000001


---

### ⚠️ PERO en la vida real:

No siempre es así, porque:

- Empresas pueden tener múltiples puntos de emisión
- Pueden iniciar secuencias desde otro número (por migración)

---

## 🧠 4. Reglas importantes del SRI

### 🔒 Se puede cambiar:

- RUC
- Código de establecimiento (una vez usado)
- Secuencias ya emitidas

- PERO HABRA UN MODAL DE ADVERTENCIA SI SE MODIFICA DATOS SENSIBLES.
---

### ✏️ Se puede configurar (antes de usar):

- Punto de emisión
- Secuencia inicial
- Tipos de documentos

---

## 🏗️ 5. Qué debe hacer tu sistema (OBLIGATORIO)

### ✔ Al crear empresa

El backend DEBE crear automáticamente:

```bash 
Empresa
└── Establecimiento:
    código: 001
    tipo: MATRIZ
    └── Punto emisión:
        código: 001
```


📌 Esto evita errores de flujo

---

## 🧾 6. Tipos de documentos electrónicos

Cada tipo tiene su propia secuencia:

- Factura
- Retención
- Nota de crédito
- Nota de débito
- Guía de remisión
- Liquidación de compra (proximamente)

---

### 📌 IMPORTANTE:

Cada documento tiene su propio contador:
Factura → 000000001
Retención → 000000001
Nota crédito → 000000001


NO comparten secuencia

---

## 🔄 7. Relación completa

```bash 
Empresa (RUC)
└── Establecimiento (001, 002...)
└── Punto emisión (001, 002...)
└── Tipo documento
└── Secuencial
```


---

## ⚠️ 8. Problema real en tu sistema

Actualmente:

❌ El sistema pide establecimiento  
❌ Pero no garantiza que exista uno usable  
❌ El usuario se pierde entre CRM y configuración  

---

## ✅ 9. Solución correcta (Backend)

### ✔ Automatizar SIEMPRE:

Al crear empresa:

```json
{
  "establecimiento": "001",
  "tipo": "MATRIZ",
  "punto_emision": "001"
}
```

# 🔐 10. Gestión de Datos Sensibles y Auditoría

Aunque el sistema automatiza la creación, el administrador debe tener control total para corregir errores de configuración inicial.

* **Logs de Auditoría:** Cualquier cambio en el RUC, Código de Establecimiento o Secuencial debe registrar: **Usuario, Fecha, Valor Anterior y Valor Nuevo.**
* **Validación de Bloqueo:** El sistema no permitirá guardar un secuencial menor al último emitido para evitar rechazos del SRI.


# 🎨 Propuesta de Flujo UX: Configuración y Emisión Fiscal

Para garantizar la coherencia entre el **CRM** y el **módulo de Facturación**, el sistema implementará un flujo de **"Protección de Integridad de Datos"**, asegurando que el usuario siempre tenga un punto de partida válido pero con la flexibilidad necesaria para ajustes administrativos.

---

## A. Vista de "Establecimientos" (Módulo de Configuración)
Esta sección es el corazón de la identidad fiscal de la empresa en el sistema.

* **Pre-configuración Automática (Zero-Setup):** Al momento de registrar una nueva empresa, el sistema genera por defecto el registro `001 - MATRIZ`. El usuario nunca encontrará esta tabla vacía, eliminando errores de referencia en el *backend*.
* **Estado de Protección (Read-Only):** Por seguridad, los campos críticos (**Código de Establecimiento, RUC y Punto de Emisión**) se presentan en estado *disabled* (bloqueados) o como texto plano para evitar ediciones accidentales durante la navegación rutinaria.
* **Acceso mediante Credenciales/Privilegios:**
    * **Trigger de Seguridad:** Al activarlo, se disparará un **Modal de Advertencia Crítica** que detalla los riesgos legales ante el SRI.
    * **Desbloqueo Logueado:** Una vez aceptado el riesgo, los *inputs* se habilitan. Cualquier cambio realizado se registrará en el historial de auditoría de la base de datos.

---

## B. Vista de "Facturación" (Módulo de Emisión)
Aquí es donde se consume la configuración previa de forma dinámica y segura.

* **Selector Inteligente de Emisión:** Un menú desplegable (*Dropdown*) permitirá elegir entre los distintos puntos de emisión configurados. Por defecto, siempre estará seleccionado el `001-001` (Matriz), agilizando la facturación para el 90% de los usuarios.
* **Gestión de Secuencial Automático:**
    * El campo del **Número de Factura** (ej. `000000045`) será de solo lectura para el usuario operativo, garantizando que la secuencia incremental no se rompa por error humano.
* **Override de Secuencia (Solo Admins):**
    * Se implementará un control de **"Ajuste de Secuencial"** protegido por permisos de Administrador.
    *







