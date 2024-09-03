# 📝 Especificación de Requisitos del API de Gestión de Pagos

## 🎯 1. **Objetivo del API**

El objetivo de este API es proporcionar una interfaz para la gestión de pagos, que permita registrar pagos, verificar su estatus, modificar dicho estatus y notificar cambios mediante un sistema de mensajería (RabbitMQ).

## ⚙️ 2. **Requisitos Funcionales**

### ✨ 2.1 **Alta de Pagos**

El API debe permitir la creación de nuevos registros de pago. Los atributos requeridos para la creación de un pago son:

- **concepto**: Una descripción breve del motivo del pago.
- **cantidad**: La cantidad numérica correspondiente al pago.
- **quién realiza el pago**: Identificación del pagador (por ejemplo, nombre o identificador único).
- **a quién se le paga**: Identificación del beneficiario del pago (por ejemplo, nombre o identificador único).
- **monto**: El monto total a pagar.
- **estatus del pago**: El estado actual del pago (ejemplo: `pendiente`, `completado`, `fallido`).

#### **Consideraciones:**
- Los atributos son obligatorios y deben ser validados antes de registrar el pago en el sistema.
- El API debe retornar un identificador único para cada pago registrado exitosamente.

### 🔍 2.2 **Verificación del Estatus del Pago**

El API debe proporcionar la capacidad de consultar el estatus actual de un pago registrado.

- **Entrada requerida**: Identificador único del pago.
- **Salida**: Estatus actual del pago.

#### **Consideraciones:**
- Si el pago no existe, el API debe retornar un error 404 indicando que el pago no fue encontrado.

### 🔄 2.3 **Cambio de Estatus del Pago**

El API debe permitir modificar el estatus de un pago existente.

- **Entrada requerida**:
  - Identificador único del pago.
  - Nuevo estatus del pago (ejemplo: `completado`, `fallido`).

#### **Consideraciones:**
- Sólo usuarios autorizados deben poder cambiar el estatus de un pago.
- El estatus debe cambiarse de manera transaccional, asegurando la integridad de la operación.

### 📩 2.4 **Notificación de Cambio de Estatus**

El API debe notificar automáticamente al sistema de mensajería (RabbitMQ) cada vez que el estatus de un pago sea modificado.

- **Evento disparador**: Cambio de estatus del pago.
- **Contenido del mensaje**: 
  - Identificador del pago.
  - Estatus nuevo.
  - Marca de tiempo del cambio.

#### **Consideraciones:**
- Si la notificación falla, debe existir un mecanismo de reintento o registro de fallos para garantizar que la notificación se procese eventualmente.

## 🔐 3. **Requisitos No Funcionales**

### 🔒 3.1 **Seguridad**
- La API debe implementarse sobre HTTPS para asegurar la transmisión segura de datos.
- Autenticación y autorización deben estar en su lugar para garantizar que sólo usuarios y sistemas autorizados puedan acceder o modificar los pagos.

### 🚀 3.2 **Rendimiento**
- La API debe ser capaz de manejar al menos 1000 solicitudes por segundo sin degradación significativa en el tiempo de respuesta.

### 📈 3.3 **Escalabilidad**
- El diseño del API debe permitir su escalabilidad horizontal, permitiendo añadir más servidores para manejar cargas crecientes.

### 🕒 3.4 **Disponibilidad**
- El API debe tener una disponibilidad del 99.9%, minimizando el tiempo de inactividad.

### ✅ 3.5 **Consistencia**
- Todas las operaciones de cambio de estatus deben ser consistentes, siguiendo un modelo transaccional que asegure la integridad de los datos.

## 🛠️ 4. **Consideraciones de Implementación**

### 🧪 4.1 **Validación de Datos**
- Todos los campos deben ser validados antes de procesar la solicitud. Campos como `monto` deben ser validados para contener valores numéricos positivos.

### ⚠️ 4.2 **Manejo de Errores**
- El API debe proporcionar mensajes de error claros y específicos, indicando la naturaleza del error (por ejemplo, `400 Bad Request` para solicitudes mal formadas, `404 Not Found` para pagos inexistentes).

### 📝 4.3 **Registro de Actividades (Logging)**
- Todas las operaciones críticas deben ser registradas, incluyendo creaciones, consultas y modificaciones de pagos, así como intentos fallidos de notificación a RabbitMQ.

## 🔗 5. **Requisitos de Integración**

### 🐇 5.1 **RabbitMQ**
- El API debe estar integrado con RabbitMQ para enviar notificaciones de cambios de estatus.
- Se deben manejar los casos de fallo en la entrega de mensajes mediante reintentos o almacenamiento en un sistema de respaldo.

### 🗄️ 5.2 **Base de Datos**
- La API debe interactuar con una base de datos relacional para almacenar los registros de pago, asegurando la integridad y consistencia de los datos.

## 👥 6. **Resumen de Roles y Responsabilidades**

- **Usuario Final**: Realiza pagos y consulta su estatus.
- **Administrador**: Modifica el estatus de pagos y revisa el sistema de notificaciones.
- **Sistema de Mensajería (RabbitMQ)**: Recibe notificaciones de cambios de estatus para su procesamiento posterior.
- 

# 📊 Diagrama de Casos de Uso

A continuación se muestra el diagrama de casos de uso para el API de gestión de pagos:

![Diagrama de Casos de Uso](https://etaili.s3.amazonaws.com/DiagramaCasoDeUsos.png)

## 📌 Notas Explicativas

### 1. **Usuario Final**
- **Descripción**: Representa a cualquier cliente o usuario del sistema que puede realizar pagos y verificar su estatus. No tiene permisos para modificar el estatus de los pagos.

### 2. **Administrador**
- **Descripción**: El Administrador es el usuario con permisos especiales para modificar el estado de los pagos. Esta acción es crítica y puede activar notificaciones automáticas al sistema de mensajería (RabbitMQ).

### 3. **RabbitMQ**
- **Descripción**: RabbitMQ es el sistema de mensajería encargado de recibir notificaciones cada vez que se cambia el estado de un pago. Este mecanismo asegura que otras partes del sistema puedan reaccionar adecuadamente a los cambios.

### 4. **Relación de Inclusión `<<include>>`**
- **Descripción**: La relación `<<include>>` entre **Cambiar Estatus del Pago** y **Notificar Cambio de Estatus** indica que cada vez que se cambia el estado de un pago, se debe notificar automáticamente al sistema de mensajería (RabbitMQ). Este es un paso obligatorio en el flujo.

### 5. **Realizar un Pago**
- **Descripción**: Realizar un Pago es el caso de uso principal del sistema, iniciando la cadena de eventos que lleva a la verificación y posible modificación del estatus del pago.

### 6. **Verificar Estatus del Pago**
- **Descripción**: La verificación del estatus permite a los usuarios finales consultar el estado actual de sus pagos, lo que es crucial para la transparencia y la satisfacción del cliente.

