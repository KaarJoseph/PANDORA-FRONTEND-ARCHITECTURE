# 🚀 PANDORA ERP - Frontend Architecture

**Arquitectura Modular por Capas (Modular Layered Architecture)**

[![Next.js](https://img.shields.io/badge/Next.js-14+-000000?style=for-the-badge&logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5+-3178C6?style=for-the-badge&logo=typescript)](https://www.typescriptlang.org/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-3+-06B6D4?style=for-the-badge&logo=tailwindcss)](https://tailwindcss.com/)
[![Zustand](https://img.shields.io/badge/Zustand-4+-orange?style=for-the-badge)](https://github.com/pmndrs/zustand)
[![TanStack Query](https://img.shields.io/badge/TanStack_Query-5+-FF4154?style=for-the-badge&logo=reactquery)](https://tanstack.com/query)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

**Realizado por:** Kaar Joseph Mashingashi Unkuch

---

## 📑 Tabla de Contenidos

- [Introducción](#-introducción)
- [Nombre de la Arquitectura](#-nombre-de-la-arquitectura)
- [Stack Tecnológico](#-stack-tecnológico)
- [Principios de Diseño](#-principios-de-diseño)
- [Estructura de Directorios](#-estructura-de-directorios)
- [Capas Internas de cada Módulo](#-capas-internas-de-cada-módulo)
- [Comunicación con el Backend](#-comunicación-con-el-backend)
- [Gestión de Estado Global](#-gestión-de-estado-global)
- [Sistema de Rutas y Protección](#-sistema-de-rutas-y-protección)
- [Consideraciones de Seguridad y Rendimiento](#-consideraciones-de-seguridad-y-rendimiento)
- [Diseño UI/UX](#-diseño-uiux)
- [Conclusión](#-conclusión)

---

## 📖 Introducción

**PANDORA** es un ERP en la nube con un modelo de negocio **multi‑tenant** donde conviven tres tipos de usuarios:

- 👑 **Super Administrador** – dueño de la plataforma FSO‑PANDORA.
- 🏢 **Administrador** – dueño de una empresa cliente (ej. Consultora).
- 👥 **Operativos** – empleados de la empresa cliente, que pueden ser:
  - 🧾 **Cajero**
  - 📋 **RRHH**
  - 📌 **General**

El objetivo de esta propuesta es definir una arquitectura de frontend **robusta, escalable y mantenible**, construida con **Next.js 14+ (App Router)** y alineada con la base de datos existente (migraciones 0001 a 0015) y con la arquitectura backend que será propuesta. La propuesta garantiza un aislamiento correcto de datos entre inquilinos (tenants), una experiencia de usuario fluida y una clara separación de responsabilidades que facilite el trabajo en equipo.

---

## 🧠 Nombre de la Arquitectura

### **Arquitectura Modular por Capas (Modular Layered Architecture)**

Este nombre refleja dos características fundamentales:

- **Modularidad** – el frontend se organiza en módulos independientes que representan cada área funcional del ERP (facturación, contabilidad, RRHH, etc.).
- **Capas** – dentro de cada módulo, el código se separa en capas con responsabilidades bien definidas (presentación, aplicación, dominio, infraestructura), lo que facilita el mantenimiento, la escalabilidad y la reutilización.

---

## 🛠️ Stack Tecnológico

| Tecnología       | Versión | Tipo                      | Descripción / Justificación                                                                                                                                                                                                                                                                                                                                                                           |
| ---------------- | ------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Next.js**      | 14+     | Framework Frontend        | ![Next.js](https://img.shields.io/badge/-Next.js-000000?style=flat&logo=next.js) Framework principal de React. Proporciona renderizado híbrido (SSR, SSG, CSR), sistema de rutas basado en archivos y optimizaciones automáticas. Ideal para un ERP porque permite páginas públicas rápidas (login, recuperación) y navegación fluida en el panel interno. El App Router facilita layouts anidados y protección de rutas. |
| **TypeScript**   | 5+      | Lenguaje                  | ![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?style=flat&logo=typescript) Extiende JavaScript con tipado estático. Permite definir interfaces alineadas con las entidades de la base de datos, evitando errores en compilación y mejorando la mantenibilidad del código en proyectos grandes.                                                                                              |
| **Tailwind CSS** | 3+      | Framework de Estilos      | ![Tailwind CSS](https://img.shields.io/badge/-Tailwind-06B6D4?style=flat&logo=tailwindcss) Framework utilitario que permite construir interfaces rápidamente sin escribir CSS personalizado. Garantiza consistencia visual y acelera el desarrollo.                                                                                                                                                      |
| **Zustand**      | 4+      | Gestión de Estado Global  | ![Zustand](https://img.shields.io/badge/-Zustand-orange?style=flat) Biblioteca ligera para estado global. Ideal para manejar autenticación, tenant activo y estado de interfaz sin la complejidad de Redux. Soporta middlewares como persistencia y devtools.                                                                                                                                          |
| **React Hook Form** | 7+   | Manejo de Formularios     | ![React Hook Form](https://img.shields.io/badge/-React_Hook_Form-EC5990?style=flat&logo=reacthookform) Biblioteca optimizada para formularios. Reduce re-renderizaciones y se integra fácilmente con validación. Fundamental para formularios complejos del ERP como facturas, empleados o contabilidad.                                                                                                  |
| **Zod**          | 3+      | Validación de Datos       | ![Zod](https://img.shields.io/badge/-Zod-3E67B1?style=flat) Permite definir esquemas de datos y validarlos en tiempo de ejecución. Garantiza que la información recibida del backend cumpla la estructura esperada. Se integra perfectamente con React Hook Form.                                                                                                                                       |
| **Axios**        | 1+      | Cliente HTTP              | ![Axios](https://img.shields.io/badge/-Axios-5A29E4?style=flat&logo=axios) Cliente para realizar peticiones al backend. Permite usar interceptores para agregar automáticamente tokens de autenticación y el identificador del tenant en cada solicitud.                                                                                                                                                |
| **TanStack Query** | 5+    | Gestión de Estado del Servidor | ![TanStack Query](https://img.shields.io/badge/-TanStack_Query-FF4154?style=flat&logo=reactquery) Biblioteca para manejar datos del servidor con cache, sincronización en segundo plano y actualizaciones automáticas. Ideal para listados dinámicos como clientes, facturas y productos.                                                                                                             |
| **date-fns**     | 3+      | Manejo de Fechas          | ![date-fns](https://img.shields.io/badge/-date--fns-FFC0CB?style=flat) Biblioteca ligera para manipulación de fechas. Necesaria para formatear fechas, manejar periodos contables y vencimientos según requisitos fiscales.                                                                                                                                                                            |
| **Recharts**     | 2+      | Gráficos y Dashboards     | ![Recharts](https://img.shields.io/badge/-Recharts-22b5bf?style=flat) Biblioteca de gráficos basada en React. Se usará para dashboards financieros, balances, evolución de ventas y reportes visuales.                                                                                                                                                                                                 |
| **NextAuth.js**  | —       | Autenticación             | ![NextAuth.js](https://img.shields.io/badge/-NextAuth.js-000000?style=flat) Sistema de autenticación para Next.js. Maneja sesiones, refresh tokens y protección de rutas, integrándose con backend basado en JWT.                                                                                                                                                                                        |
| **Lucide React** | —       | Iconos                    | ![Lucide](https://img.shields.io/badge/-Lucide-F56565?style=flat&logo=lucide) Biblioteca de iconos vectoriales moderna y consistente para mejorar la interfaz del usuario.                                                                                                                                                                                                                             |
| **shadcn/ui**    | —       | Componentes UI            | ![shadcn/ui](https://img.shields.io/badge/-shadcn/ui-000000?style=flat) Colección de componentes reutilizables basados en Radix UI y Tailwind. Permite copiar componentes al proyecto y personalizarlos completamente, acelerando el desarrollo.                                                                                                                                                       |

---

## 🎯 Principios de Diseño

- **🧩 Modularidad por funcionalidad** – Cada funcionalidad del sistema se organiza en módulos independientes dentro de `modules/`. Permite trabajo en paralelo, pruebas aisladas y mejor organización.
- **📐 Separación de responsabilidades en capas** – Dentro de cada módulo se distinguen cuatro capas: **types**, **services**, **hooks**, **components**. Esto facilita el mantenimiento y la escalabilidad.
- **🏢 Multi-tenancy en el frontend** – El tenant activo se obtiene del contexto de autenticación y se inyecta en cada petición mediante un interceptor de Axios. Las rutas y datos siempre se filtran por tenant.
- **🔒 Protección por roles** – Las rutas y componentes se protegen según el rol del usuario (Super Admin, Admin, operativos). Se utiliza middleware de Next.js y HOCs.
- **♻️ Reutilización de componentes** – Biblioteca de componentes UI reutilizables en `components/` (botones, inputs, tablas) y componentes compuestos (formularios de dirección, selectores de impuestos). Garantiza consistencia visual.
- **⚡ Experiencia de usuario fluida** – Se aprovechan las capacidades de Next.js (prefetching, SSR para páginas públicas) y TanStack Query (caché y revalidación en segundo plano) para ofrecer navegación rápida y datos actualizados.
- **📈 Escalabilidad y mantenibilidad** – Organización modular, tipado fuerte con TypeScript y validación con Zod permiten que el proyecto crezca de forma ordenada y facilita la incorporación de nuevos desarrolladores.

---

## 📂 Estructura de Directorios

La siguiente estructura de carpetas es la base del proyecto. Cada carpeta y archivo tiene un propósito definido.

```text
pandora-frontend/
├── 📄 .env.example # Variables de entorno de ejemplo
├── 📄 .eslintrc.json # Configuración de ESLint
├── 📄 .prettierrc # Configuración de Prettier
├── 📄 next.config.js # Configuración de Next.js
├── 📄 tailwind.config.js # Configuración de Tailwind CSS
├── 📄 tsconfig.json # Configuración de TypeScript
├── 📄 package.json # Dependencias y scripts
├── 📁 public/ # Archivos estáticos
│ ├── favicon.ico
│ ├── 📁 logos/ # Logos de tenants (cargados dinámicamente)
│ └── 📁 images/ # Imágenes globales (logo de FSO-PANDORA, etc.)
│
└── 📁 src/
├── 📁 app/ # App Router de Next.js (rutas y layouts)
│ ├── 📁 (auth)/ # Grupo de rutas públicas (sin autenticación)
│ │ ├── 📁 login/
│ │ │ └── 📄 page.tsx # Página de inicio de sesión
│ │ ├── 📁 recuperar/
│ │ │ └── 📄 page.tsx # Página de recuperación de contraseña
│ │ └── 📄 layout.tsx # Layout para las páginas de autenticación
│ │
│ ├── 📁 (dashboard)/ # Grupo de rutas protegidas (requieren autenticación)
│ │ ├── 📁 superadmin/ # Rutas exclusivas de Super Administrador
│ │ │ ├── 📁 tenants/
│ │ │ │ └── 📄 page.tsx # Listado de empresas clientes (tenants)
│ │ │ ├── 📁 tenants/[id]/
│ │ │ │ └── 📄 page.tsx # Detalle de un tenant
│ │ │ ├── 📁 monitoreo/
│ │ │ │ └── 📄 page.tsx # Dashboard global
│ │ │ └── 📄 layout.tsx # Layout específico para Super Admin
│ │ │
│ │ ├── 📁 admin/ # Rutas de Administrador de empresa
│ │ │ ├── 📁 configuracion/
│ │ │ │ └── 📄 page.tsx # Configuración de la empresa
│ │ │ ├── 📁 establecimientos/
│ │ │ │ └── 📄 page.tsx # Gestión de establecimientos
│ │ │ ├── 📁 roles/
│ │ │ │ └── 📄 page.tsx # Creación y edición de roles operativos
│ │ │ ├── 📁 usuarios/
│ │ │ │ └── 📄 page.tsx # Creación de usuarios operativos
│ │ │ ├── 📁 productos/
│ │ │ │ └── 📄 page.tsx # Gestión de productos/servicios
│ │ │ ├── 📁 clientes/
│ │ │ │ └── 📄 page.tsx # Gestión de clientes
│ │ │ ├── 📁 reportes/
│ │ │ │ └── 📄 page.tsx # Reportes gerenciales
│ │ │ └── 📄 layout.tsx # Layout específico para Administradores
│ │ │
│ │ ├── 📁 cajero/ # Rutas para operativos tipo Cajero
│ │ │ ├── 📁 punto-venta/
│ │ │ │ └── 📄 page.tsx # Interfaz principal de punto de venta
│ │ │ ├── 📁 historial/
│ │ │ │ └── 📄 page.tsx # Historial de ventas del día
│ │ │ ├── 📁 cierres/
│ │ │ │ └── 📄 page.tsx # Cierre de caja
│ │ │ └── 📄 layout.tsx # Layout específico para Cajeros
│ │ │
│ │ ├── 📁 rrhh/ # Rutas para operativos tipo RRHH
│ │ │ ├── 📁 empleados/
│ │ │ │ └── 📄 page.tsx # Gestión de empleados
│ │ │ ├── 📁 nominas/
│ │ │ │ └── 📄 page.tsx # Gestión de nóminas
│ │ │ ├── 📁 prestamos/
│ │ │ │ └── 📄 page.tsx # Gestión de préstamos a empleados
│ │ │ ├── 📁 vacaciones/
│ │ │ │ └── 📄 page.tsx # Solicitudes y aprobaciones de vacaciones
│ │ │ └── 📄 layout.tsx # Layout específico para RRHH
│ │ │
│ │ ├── 📁 general/ # Rutas para operativos tipo General
│ │ │ ├── 📁 clientes/
│ │ │ │ └── 📄 page.tsx # Gestión de clientes
│ │ │ ├── 📁 productos/
│ │ │ │ └── 📄 page.tsx # Gestión de productos
│ │ │ ├── 📁 reportes/
│ │ │ │ └── 📄 page.tsx # Reportes básicos
│ │ │ └── 📄 layout.tsx # Layout específico para Generales
│ │ │
│ │ └── 📄 layout.tsx # Layout principal del dashboard (con sidebar y header)
│ │
│ ├── 📁 api/ # API routes de Next.js (solo para integración con NextAuth)
│ │ └── 📁 auth/[...nextauth]/
│ │ └── 📄 route.ts # Manejador de NextAuth
│ │
│ └── 📄 layout.tsx # Layout raíz (html, body, providers)
│
├── 📁 components/ # Componentes compartidos
│ ├── 📁 ui/ # Componentes atómicos (basados en shadcn/ui o propios)
│ │ ├── 📁 Button/
│ │ ├── 📁 Input/
│ │ ├── 📁 Table/
│ │ ├── 📁 Dialog/
│ │ ├── 📁 Select/
│ │ ├── 📁 Card/
│ │ ├── 📁 Alert/
│ │ ├── 📁 Badge/
│ │ └── 📄 index.ts
│ │
│ ├── 📁 forms/ # Componentes de formulario reutilizables
│ │ ├── 📁 RucInput/
│ │ ├── 📁 CedulaInput/
│ │ ├── 📁 FechaInput/
│ │ ├── 📁 ImpuestoSelect/
│ │ ├── 📁 FormaPagoSelect/
│ │ ├── 📁 MonedaInput/
│ │ ├── 📁 TelefonoInput/
│ │ └── 📄 index.ts
│ │
│ └── 📁 layouts/ # Componentes de layout compartidos
│ ├── 📁 Sidebar/
│ ├── 📁 Header/
│ └── 📁 PageHeader/
│
├── 📁 modules/ # Módulos de negocio (cada uno con su estructura interna)
│ ├── 📁 billing/ # Facturación
│ │ ├── 📁 components/
│ │ ├── 📁 hooks/
│ │ ├── 📁 services/
│ │ ├── 📁 types/
│ │ └── 📄 index.ts
│ ├── 📁 accounting/ # Contabilidad
│ ├── 📁 inventory/ # Inventario
│ ├── 📁 rrhh/ # Recursos Humanos
│ ├── 📁 treasury/ # Tesorería
│ ├── 📁 sales/ # Ventas
│ ├── 📁 fixed-assets/ # Activos Fijos
│ └── 📁 reports/ # Reportes
│
├── 📁 core/ # Funcionalidad transversal
│ ├── 📁 auth/ # Autenticación y autorización
│ ├── 📁 config/ # Configuración global (axios, constantes)
│ ├── 📁 lib/ # Utilidades generales (formatters, validators)
│ └── 📁 types/ # Tipos globales
│
├── 📁 stores/ # Estado global con Zustand
│ ├── 📄 authStore.ts
│ ├── 📄 tenantStore.ts
│ ├── 📄 uiStore.ts
│ ├── 📄 cajaStore.ts
│ └── 📄 index.ts
│
├── 📁 styles/ # Estilos globales
│ └── 📄 globals.css
│
├── 📄 middleware.ts # Middleware de Next.js para protección de rutas
│
└── 📄 providers.tsx # Proveedores globales (ThemeProvider, AuthProvider, QueryClientProvider)
```


---

## 🧱 Capas Internas de cada Módulo

Dentro de cada módulo de negocio (por ejemplo `billing`, `accounting`, `rrhh`, etc.), el código se organiza en **cuatro capas** bien definidas. Cada capa tiene una responsabilidad única y se comunica con las demás de forma controlada.

```text
┌─────────────────────────────────────────────────────────────┐
│ PÁGINA (app/) │
│ (orquesta hooks y componentes) │
└───────────────────────────────┬─────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────┐
│ CAPA DE APLICACIÓN (hooks/) │
│ - Custom hooks que usan servicios y TanStack Query │
│ - Lógica de negocio reutilizable │
└───────────────────────────────┬─────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────┐
│ CAPA DE SERVICIOS (services/) │
│ - Funciones que realizan peticiones HTTP al backend │
│ - Retornan promesas tipadas │
└───────────────────────────────┬─────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────┐
│ CAPA DE TIPOS (types/) │
│ - Interfaces TypeScript y esquemas Zod │
│ - Contrato entre frontend y backend │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────┐
│ CAPA DE PRESENTACIÓN (components) │
│ - Componentes React "tontos" │
│ - Reciben datos y callbacks │
└─────────────────────────────────────┘
```

**Flujo de datos:**

1. La página (en `app/`) importa un hook (ej. `useInvoices`).
2. El hook llama al servicio (`invoiceService.getInvoices`) y gestiona el estado con TanStack Query.
3. El servicio realiza la petición HTTP mediante Axios (con interceptores de autenticación y tenant).
4. Los datos retornados pasan por el hook y se entregan a la página.
5. La página renderiza un componente de presentación (ej. `InvoiceTable`) pasándole los datos y funciones.
6. Al interactuar (ej. clic en "Crear factura"), la página invoca una mutación (`useCreateInvoice`) que llama al servicio correspondiente y, al completarse, invalida las consultas para refrescar la lista.

---

## 🌐 Comunicación con el Backend

### ¿Cómo se comunican?

Toda la comunicación se realiza mediante peticiones HTTP (REST) con **Axios**. Cada petición debe incluir:

- 🔑 Token de autenticación (JWT) en el header `Authorization`.
- 🏢 Identificador del tenant (slug o ID) en un header personalizado, ej. `X-Tenant-Slug`.

Para no repetir estos headers en cada llamada, configuramos un **interceptor de Axios** a nivel global (`core/config/axios.ts`) que añade automáticamente el token y el tenant obtenidos del store de Zustand.

### Organización de servicios para 108 tablas

Con 108 tablas en la base de datos, la estrategia es:

- **Agrupar por módulos de negocio**: Cada módulo agrupa las tablas relacionadas.
- **Un archivo de servicio por recurso principal**: Por ejemplo, `invoiceService.ts`, `creditNoteService.ts`, etc.
- **Métodos CRUD estándar**: `getAll`, `getById`, `create`, `update`, `delete`.
- **Tipado fuerte**: Todos los métodos reciben y devuelven datos tipados con las interfaces de la capa `types`.

**Ejemplo para el módulo de facturación:**

```text
modules/billing/
├── types/
│ ├── invoice.types.ts
│ ├── receivable.types.ts
│ └── collection.types.ts
├── services/
│ ├── invoiceService.ts
│ ├── receivableService.ts
│ └── collectionService.ts
├── hooks/
│ ├── useInvoices.ts
│ ├── useInvoice.ts
│ ├── useCreateInvoice.ts
│ ├── useReceivables.ts
│ └── useRegisterCollection.ts
└── components/
├── InvoiceTable/
├── InvoiceForm/
├── ReceivableTable/
└── CollectionModal/
```

---

## 🗃️ Gestión de Estado Global

Los stores se implementan con **Zustand** y mantienen información accesible desde cualquier componente.

| Store          | Descripción                                                                 |
|----------------|-----------------------------------------------------------------------------|
| `authStore`    | Usuario autenticado (id, nombre, email, rol, tipo_operativo), tokens.      |
| `tenantStore`  | Tenant activo (id, slug, nombre, plan, configuración).                      |
| `uiStore`      | Estado de la interfaz: tema, sidebar abierto/cerrado, título de página.    |
| `cajaStore`    | (Opcional) Estado de la caja actual para el rol de cajero.                  |

**Persistencia:** Algunos stores (como `authStore` y `tenantStore`) pueden persistirse en `localStorage` usando el middleware `persist` de Zustand, manteniendo la sesión al recargar la página.

**Ejemplo de store:**

```ts
// stores/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AuthState {
  user: User | null;
  token: string | null;
  login: (credentials) => Promise<void>;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      login: async (credentials) => { /* ... */ },
      logout: () => set({ user: null, token: null }),
    }),
    { name: 'auth-storage' }
  )
);
```
## 🛡️ Sistema de Rutas y Protección por Roles

### Estructura de rutas en `app/`

- **(auth):**  
  Rutas públicas del sistema.  
  Incluyen páginas como:
  - `login`
  - `recuperar contraseña`

- **(dashboard):**  
  Rutas protegidas que comparten un layout común con **sidebar** y **header**.

  Dentro de este grupo se organizan subcarpetas según el rol del usuario:

  - `superadmin/`
  - `admin/`
  - `cajero/`
  - `rrhh/`
  - `general/`

---

### Middleware de protección

El archivo **`middleware.ts`** se ejecuta antes de cada petición y tiene como función:

- Verificar si el usuario está **autenticado**.
- Validar el **rol del usuario**.
- Redirigir al usuario si intenta acceder a una ruta que no le corresponde.

Este control se realiza utilizando **NextAuth.js** para gestionar las sesiones y la autenticación.



```ts
// middleware.ts (ejemplo conceptual)
export default withAuth(
  function middleware(req) {
    const token = req.nextauth.token;
    const path = req.nextUrl.pathname;

    if (path.startsWith('/superadmin') && token?.role !== 'super_admin') {
      return NextResponse.redirect(new URL('/dashboard', req.url));
    }
    // ... similar para otros roles
  },
  { callbacks: { authorized: ({ token }) => !!token } }
);
```

### Protección a nivel de componente

Además del middleware, se pueden utilizar **HOCs (Higher-Order Components)** como `withRole` para restringir el acceso a partes específicas de una página o componente.

---

## 🔒 Consideraciones de Seguridad y Rendimiento

### Seguridad

- **Tokens JWT en memoria:**  
  Los tokens se almacenan en memoria (no en `localStorage`) para evitar ataques **XSS**.  
  Si se requiere persistencia, se utiliza un **refresh token** almacenado en una **cookie httpOnly**.

- **Interceptores de Axios:**  
  Se añaden **headers personalizados** automáticamente en cada petición mediante interceptores.

- **Validación de datos:**  
  Se utiliza **Zod** para validar tanto las respuestas del backend como los datos enviados desde formularios.

- **Protección CSRF:**  
  Configuración de cookies con `sameSite` y uso de **tokens CSRF** cuando sea necesario.

---

### Rendimiento

- **Caché con TanStack Query:**  
  Reduce la cantidad de peticiones al servidor y mejora la velocidad de respuesta de la aplicación.

- **Code Splitting automático:**  
  Next.js divide automáticamente el código por páginas, reduciendo el tamaño inicial de carga.

- **Lazy Loading:**  
  Los módulos pesados se cargan de forma diferida utilizando **dynamic imports**.

- **Optimización de imágenes:**  
  Uso del componente `next/image` para mejorar la carga y optimización de imágenes.

- **Virtualización de tablas:**  
  Para listados largos se utilizarán librerías como:
  - `react-virtualized`
  - `@tanstack/virtual`

---

## 🎨 Diseño UI/UX

### Principios de Diseño Visual

- **Consistencia:**  
  Uso de un **sistema de diseño unificado** con componentes reutilizables (`shadcn/ui`) y **tokens de diseño** como colores, tipografía y espaciado.

- **Jerarquía visual:**  
  Diferenciación clara entre elementos mediante **tipografía, color y espaciado**.

- **Feedback inmediato:**  
  Indicadores visibles de estados del sistema como:
  - cargas (*loaders*)
  - errores
  - confirmaciones (*toasts*)

- **Accesibilidad:**  
  Cumplimiento de estándares **WCAG 2.1**, incluyendo:
  - contrastes adecuados
  - etiquetas **ARIA**
  - navegación por teclado

- **Modo oscuro:**  
  Soporte nativo de **dark mode** utilizando **Tailwind CSS** y un contexto de tema.

## 🧩 Componentes UI

**Biblioteca base:**  
Se utilizará **shadcn/ui** como base de componentes, personalizada con los colores y estilos de la marca.

### Componentes clave

- **Tablas avanzadas**
  - Ordenamiento
  - Filtrado
  - Paginación

- **Formularios**
  - Validación en tiempo real
  - Manejo eficiente del estado del formulario

- **Modales y diálogos**
  - Utilizados para acciones críticas como eliminaciones, confirmaciones o edición de datos sensibles.

- **Gráficos interactivos**
  - Implementados con **Recharts** para dashboards y reportes.

- **Estados de carga**
  - Uso de **loaders** y **skeletons** para mejorar la percepción de rendimiento.

---

## 👤 Experiencia de Usuario (UX)

- **Onboarding guiado**
  - Tours interactivos para nuevos usuarios que explican las principales funcionalidades del sistema.

- **Atajos de teclado**
  - Acciones rápidas como:
  - guardar
  - imprimir
  - navegar entre módulos

- **Navegación persistente**
  - Uso de **breadcrumbs** y **menús contextuales** para facilitar la orientación dentro del sistema.

- **Filtros avanzados**
  - Listados con múltiples criterios de filtrado.
  - Posibilidad de **guardar vistas personalizadas**.

- **Responsive Design**
  - Adaptación a **dispositivos móviles y tablets**, aunque el sistema esté optimizado principalmente para **desktop**.

---

## 🛠️ Herramientas de Diseño

- **Figma**
  - Diseño de interfaces y prototipado de pantallas.

- **Storybook**
  - Desarrollo y documentación de componentes de forma aislada.

- **Chromatic**
  - Pruebas visuales automáticas y revisión de componentes.

---

## ✅ Conclusión

La **Arquitectura Modular por Capas** presentada proporciona una base sólida para el desarrollo del frontend de **PANDORA**.  
Al estar alineada con la estructura del sistema y con las necesidades de **multi-tenancy** y **roles**, permite construir una plataforma robusta y escalable.

### Beneficios principales

- **Escalabilidad**  
  Los módulos independientes permiten añadir nuevas funcionalidades sin afectar las existentes.

- **Mantenibilidad**  
  La separación en capas facilita la localización de errores y la incorporación de nuevos desarrolladores.

- **Seguridad**  
  La protección de rutas y la validación de datos ayudan a minimizar riesgos.

- **Rendimiento**  
  Las técnicas de caché y optimización mejoran la velocidad y la eficiencia del sistema.

- **Experiencia de usuario**  
  Un diseño UI/UX centrado en el usuario y consistente visualmente mejora la productividad y la satisfacción del usuario final.
