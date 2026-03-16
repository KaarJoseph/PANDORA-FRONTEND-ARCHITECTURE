# 🚀 Pandora ERP - Flujo de Sistema y Jerarquía de Usuarios

Este documento define la **lógica de aprovisionamiento**, el **proceso de onboarding** y la **gestión de permisos** del ecosistema **Pandora**.

---

# 🧩 1. Definición de Planes (Configurables)

El sistema cuenta con **tres niveles de servicio**.  
Los **costos, límites de usuarios y módulos** están **parametrizados en la base de datos** y cuentan con una **interfaz en el panel de Super Admin para su ajuste**.

### 📦 Planes Disponibles

| Plan | Descripción |
|-----|-----|
| 🧾 **FACTURACIÓN** | Plan base |
| ⚙️ **PANDORA** | Plan intermedio |
| 🏢 **PANDORA ERP** | Plan integral |

---

# 🏗️ 2. Flujo de Creación: Empresa y Usuario Inicial

El registro está diseñado para ser **rápido**, delegando la **carga de datos pesados al usuario final**.

---

## 👑 Fase 1: Registro por Super Admin

El **Super Admin** da de alta a un nuevo cliente con la siguiente **información mínima**.

### 🏢 Datos de Empresa

- Nombre comercial  
- RUC  
- Razón social  

### ⚙️ Configuración de Plan

- Selección del plan contratado  
- Fecha de activación  
- Fecha de expiración  

### 👤 Usuario Administrador

- Selección de tipo de usuario  
- Email  
- Asignación de la Empresa creada  

---

## ⚙️ Fase 2: Generación de Acceso (Back-end)

El sistema genera automáticamente una **contraseña temporal**.

📧 Se envía un **correo electrónico al usuario** con sus **credenciales de acceso**.

---

# 🔑 3. Flujo de Onboarding (Primer Inicio de Sesión)

Cuando el usuario ingresa **por primera vez** con su **clave temporal**, el sistema activa un **asistente de configuración**.

---

## 👤 Perfil de Usuario

El sistema **redirige forzosamente** a una ventana para completar:

- Nombres  
- Apellidos  
- Teléfono  
- Cédula  

---

## 🏢 Configuración de Empresa (Solo Administrador Propietario)

- Ingreso de **Dirección Matriz**
- Carga de **Logo**
- Carga de **Firma Electrónica**  
  *(Archivo `.p12` + Contraseña)*

---

## ✅ Finalización

Tras completar la **legalización y datos de perfil**, el sistema **desbloquea el Dashboard principal**.

---

# 💳 4. Gestión de Pagos y Suspensión

Para mantener el sistema **activo**, el cliente debe **gestionar su vigencia**.

---

## 📤 Carga de Comprobante

Existe un apartado donde el cliente **sube su comprobante de pago** para **validación manual de nuestra parte**.

---

## 📅 Vigencia

| Tipo | Duración |
|-----|-----|
| 📆 Mensual | 30 días |
| 🗓️ Anual | 365 días |

---

## ⏳ Días de Gracia

| Plan | Días |
|-----|-----|
| 📆 Mensual | 10 días |
| 🗓️ Anual | 30 días |

Antes de la **suspensión automática**.

---

# 👥 5. Jerarquía y Permisos de Usuarios

---

## 🧭 Niveles de Mando

### 👑 Super Administrador (Propietario)

- Máximo nivel  
- No puede ser eliminado por otros **Super Admins**

---

### 🏢 Administrador (Propietario)

- Cliente principal de la empresa  
- Tiene **control total sobre su instancia**
- No puede ser eliminado por otros administradores que él mismo cree

---

### 👨‍💻 Operativos

- Empleados de la empresa

---

# 🔐 Restricciones de Usuario y Módulos

---

## 📊 Límites por Plan

El **Super Admin** puede **aumentar manualmente la cantidad de usuarios permitidos** si la empresa lo solicita.

Ejemplo:

- En el plan **Facturación**, el límite base es de **3 usuarios**, pero es **ampliable**.

---

## ⚙️ Configuración de Operativos

Al crear un operativo, solo se requiere:

📧 **Email**

---

### 📌 Permisos por Defecto

Por defecto:

- Solo visualizan el **módulo de Facturación**

---

### 🔧 Configuración por Administrador

El **Administrador** puede habilitar **ventanas adicionales** mediante **Checks de Permisos** para que el operativo acceda a otros módulos según su rol.

---

# 🧠 Notas Técnicas

---

## 🔒 Seguridad

La **contraseña temporal debe ser cambiada obligatoriamente** tras el **primer login exitoso**.

---

## 🧾 Límite de Facturación

Todos los planes cuentan con **facturación electrónica ilimitada**.

---

# 📊 Visualización General del Flujo

```text
👑 Super Admin
      │
      ▼
🏢 Crea Empresa + Admin
      │
      ▼
⚙️ Sistema genera contraseña temporal
      │
      ▼
📧 Usuario recibe correo
      │
      ▼
🔑 Primer Login
      │
      ▼
🚀 Onboarding obligatorio
      │
      ├── 👤 Completar perfil
      │
      └── 🏢 Configurar empresa
             │
             ▼
        📊 Dashboard activo
