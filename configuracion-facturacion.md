# 📄 Configuración de Empresa y Facturación Electrónica

Este documento define la configuración necesaria para la empresa dentro del sistema, incluyendo reglas tributarias y parámetros para la emisión de comprobantes electrónicos.

## 🏢 1. Datos Básicos de la Empresa

| Campo | Tipo | Descripción |
|-------|------|-------------|
| RUC | String | Identificador tributario único |
| Razón Social | String | Nombre legal registrado en el SRI |
| Nombre Comercial | String | Nombre comercial de la empresa |
| **Logo** | **url** | **Imagen corporativa (opcional, para plantillas)** |
| Número de Establecimientos | String | Formato: 001, 002, etc. |
| Matriz (opcional) | String | Indica la dirección de la matriz |
| Obligado a llevar contabilidad | Boolean | true = Sí, false = No |

💡 *Este bloque permite estructurar la empresa y soportar sucursales a futuro.*

## 🧾 2. Clasificación Tributaria

> **⚠️ IMPORTANTE:** Esta configuración afecta directamente al cálculo de impuestos.

| Campo | Tipo | Descripción | Implementación |
|-------|------|-------------|----------------|
| Contribuyente Especial | Boolean | Clasificación del SRI | Checkbox |
| Gran Contribuyente | Boolean | Clasificación avanzada | Checkbox |
| Exportador | Enum | NO_APLICA / HABITUAL / NO_HABITUAL | Select (sin lógica por ahora) |
| Agente de Retención | Boolean | Indica si aplica retenciones | Checkbox (sin lógica por ahora) |

⚠️ **Nota:**  
Estos campos se almacenan, pero su lógica no será implementada en esta fase.

## 💰 3. Régimen RIMPE

Define el comportamiento del IVA en la facturación.

| Tipo | Descripción | IVA |
|------|-------------|-----|
| NEGOCIO_POPULAR | Contribuyente de bajo ingreso | 0% |
| RIMPE | Régimen general RIMPE | 15% |

🔥 **Regla de Negocio**  
- Si **NEGOCIO_POPULAR** → IVA = 0%  
- Si **RIMPE** → IVA = 15%  

⚠️ **CRÍTICO:**  
Una mala configuración genera facturación incorrecta y riesgo tributario.

## 📍 4. Información de Contacto

| Campo | Tipo | Descripción |
|-------|------|-------------|
| Ciudad | String | Ubicación de la empresa |
| Teléfono | String | Número de contacto |
| Dirección | String | Dirección fiscal |
| Email | String | Correo del propietario / administrador |

💡 *Se utilizará el correo del usuario administrador (no institucional).*

## 🔐 5. Firma Electrónica

Configuración necesaria para la firma de comprobantes electrónicos.

### 📁 Certificado

| Campo | Tipo | Descripción |
|-------|------|-------------|
| Archivo Firma | File (.p12 / .pfx) | Certificado digital |
| Estado | Enum | VALIDO / INVALIDO |

### 🔑 Clave de Firma

| Campo | Tipo | Descripción |
|-------|------|-------------|
| Clave | String (segura) | Contraseña del certificado |

🔒 **Validación de Clave**  
- Se valida al momento de subir el certificado  
- Si la clave es incorrecta → no se guarda  
- ⚠️ La clave no debe almacenarse en texto plano.

### 📅 Expiración de Firma

| Campo | Tipo | Descripción |
|-------|------|-------------|
| Fecha Expiración | DateTime | Fecha de vencimiento del certificado |

⚙️ **Estrategia de Validación (DECISIÓN ARQUITECTÓNICA)**

❌ NO validar la firma al momento de facturar  
(para evitar latencia y sobrecarga)

✅ Validación mediante proceso programado (CRON JOB)  
- Frecuencia: diaria (recomendado)  
- Acción:  
  - Verificar fecha de expiración  
  - Marcar estado como INVALIDO si aplica  
  - Notificar al usuario

🔔 **Notificaciones**  
- Firma próxima a expirar  
- Firma expirada

⚙️ **Uso en Facturación**  
Flujo optimizado:  
1. Se genera el XML  
2. Se obtiene el certificado almacenado  
3. Se usa la clave previamente validada  
4. Se firma el documento  
5. Se envía al SRI  

🚀 *Sin validaciones pesadas en tiempo real*

## 🧾 6. Establecimientos y Puntos de Emisión

💡 *Este módulo define cómo se generan los números de comprobantes electrónicos.*

### 📍 Estructura

| Campo | Tipo | Descripción |
|-------|------|-------------|
| Establecimiento | String | Código de sucursal (001) |
| Punto de Emisión | String | Código interno (001) |

👉 **Formato final de documentos:**  
`001-001`

### 📄 Tipos de Documento (Configuración Base)

| Documento | ¿Requerido? | Notas |
|-----------|-------------|-------|
| Factura | ✅ Sí | Principal |
| Retención | ⚠️ Opcional | Fase futura |
| Liquidación de Compra | ⚠️ Opcional | Fase futura |

💡 *En esta fase se recomienda implementar únicamente Factura.*

### 🔢 Secuencia de Documentos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| Secuencia Inicial | Number | Número inicial de documentos |

- Incremento automático  
- Control interno de numeración

## 🚨 Validaciones Críticas

- ✔ RUC obligatorio  
- ✔ Régimen RIMPE obligatorio  
- ✔ Firma cargada correctamente  
- ✔ Clave validada previamente  
- ✔ Estructura válida (XXX-XXX)

## ⚠️ Riesgos Identificados

- ❌ Configuración incorrecta de RIMPE → IVA erróneo  
- ❌ Firma expirada sin control → rechazo del SRI  
- ❌ Mala configuración de establecimiento → errores en documentos

## 🚀 Consideraciones Futuras

- Módulo completo de retenciones  
- Módulo de exportadores  
- Multi-establecimientos avanzado  
- Renovación de firma electrónica  
- Gestión segura de credenciales (vault)

## ✅ Conclusión

El sistema se basa en tres pilares:

- 🧾 **Régimen RIMPE** → define impuestos  
- 🔐 **Firma electrónica** → valida documentos  
- 🏢 **Establecimientos** → estructuran la emisión

La arquitectura prioriza:

- ⚡ Rendimiento (sin validaciones en tiempo real)  
- 🔒 Seguridad  
- 📈 Escalabilidad
