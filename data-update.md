# Actualizaciones de Arquitectura de Base de Datos y Lógica de Super Admin

Este documento detalla los cambios necesarios para la gestión de usuarios de alto nivel y la creación de la entidad Super Admin, enfocándose en la jerarquía de privilegios y la simplificación de parámetros de inicialización.

## 1. Actualización de Parámetros: `createSuperAdmin`

Se debe refactorizar el método de creación de Super Admins para que sea más conciso. La estructura de datos de entrada (DTO) ahora se limitará estrictamente a la identidad y la vinculación empresarial básica.

**Nuevos Parámetros Requeridos:**
* `email`: Identificador único de acceso.
* `password`: Credencial de autenticación (debe ser hasheada antes de la persistencia).
* `razonSocialId`: Vinculación obligatoria con la entidad legal/empresa correspondiente.

---

