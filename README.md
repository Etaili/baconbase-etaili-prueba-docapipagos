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
  
---

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

# 🖼️ Diagrama de Arquitectura

A continuación se muestra el diagrama de arquitectura del API de gestión de pagos:

![Diagrama de Arquitectura](https://etaili.s3.amazonaws.com/Diagrama+de+Arquitectura.png)

## 📌 Componentes Principales

### 1. **Cliente (Frontend)**
- **Descripción**: La interfaz de usuario desde la cual los clientes interactúan con el sistema, ya sea para realizar pagos o consultar su estatus.

### 2. **API Gateway**
- **Descripción**: El punto de entrada para todas las solicitudes al sistema. Este componente dirige las solicitudes al servicio adecuado.

### 3. **Servicio de Pagos (Backend)**
- **Descripción**: Maneja toda la lógica de negocio relacionada con la gestión de pagos, incluyendo la creación, verificación, y modificación de pagos.

### 4. **Base de Datos**
- **Descripción**: Almacena todos los registros de pago, asegurando la persistencia y consistencia de los datos.

### 5. **RabbitMQ**
- **Descripción**: Sistema de mensajería encargado de recibir y gestionar las notificaciones cuando el estatus de un pago cambia.

---

# 📚 Documentación del API de Gestión de Pagos

## 🔗 Endpoints del API

### 1. **Crear un Pago**

- **Endpoint**: `POST /api/pagos`
- **Descripción**: Este endpoint permite la creación de un nuevo pago en el sistema.

**Parámetros de Entrada**:

| Parámetro         | Tipo     | Descripción                                       | Obligatorio |
|-------------------|----------|---------------------------------------------------|-------------|
| `concepto`        | `string` | Descripción breve del motivo del pago.            | Sí          |
| `cantidad`        | `number` | La cantidad numérica correspondiente al pago.     | Sí          |
| `quien_realiza`   | `string` | Identificación del pagador (nombre o ID).         | Sí          |
| `a_quien_se_paga` | `string` | Identificación del beneficiario (nombre o ID).    | Sí          |
| `monto`           | `number` | Monto total a pagar.                              | Sí          |
| `estatus`         | `string` | Estado inicial del pago (`pendiente`).            | Sí          |

**Respuesta**:

| Campo          | Tipo     | Descripción                               |
|----------------|----------|-------------------------------------------|
| `id_pago`      | `string` | Identificador único del pago creado.       |
| `concepto`     | `string` | Descripción del motivo del pago.           |
| `cantidad`     | `number` | La cantidad numérica correspondiente.      |
| `quien_realiza`| `string` | Identificación del pagador.                |
| `a_quien_se_paga`| `string`| Identificación del beneficiario.           |
| `monto`        | `number` | Monto total a pagar.                       |
| `estatus`      | `string` | Estado actual del pago (`pendiente`).      |
| `fecha_creacion`| `string` | Fecha y hora en que se creó el pago.      |

**Ejemplo de Solicitud**:

```http
POST /api/pagos
```

```json
{
  "concepto": "Compra de Producto X",
  "cantidad": 2,
  "quien_realiza": "Juan Perez",
  "a_quien_se_paga": "Compañía XYZ",
  "monto": 500.00,
  "estatus": "pendiente"
}
```
**Ejemplo de Respuesta**:

```json
{
  "id_pago": "123456789",
  "concepto": "Compra de Producto X",
  "cantidad": 2,
  "quien_realiza": "Juan Perez",
  "a_quien_se_paga": "Compañía XYZ",
  "monto": 500.00,
  "estatus": "pendiente",
  "fecha_creacion": "2024-09-03T12:34:56Z"
}
```

### 2. **Verificar Estatus de un Pago**

- **Endpoint**: `GET /api/pagos/{id}/estatus`
- **Descripción**: Este endpoint permite consultar el estatus actual de un pago registrado.

**Parámetros de Entrada**:

| Parámetro | Tipo     | Descripción                       | Obligatorio |
|-----------|----------|-----------------------------------|-------------|
| `id`      | `string` | Identificador único del pago.     | Sí          |

**Respuesta**:

| Campo               | Tipo     | Descripción                     |
|---------------------|----------|---------------------------------|
| `id_pago`           | `string` | Identificador único del pago.   |
| `estatus`           | `string` | Estado actual del pago.         |
| `fecha_actualizacion` | `string` | Fecha y hora de la última actualización del estatus. |

**Ejemplo de Solicitud**:

```http
GET /api/pagos/123456789/estatus
```

**Ejemplo de Respuesta**:

```json
{
  "id_pago": "123456789",
  "estatus": "pendiente",
  "fecha_actualizacion": "2024-09-03T12:34:56Z"
}
```

### 3. **Cambiar Estatus de un Pago**

- **Endpoint**: `PUT /api/pagos/{id}/estatus`
- **Descripción**: Este endpoint permite modificar el estatus de un pago existente.

**Parámetros de Entrada**:

| Parámetro | Tipo     | Descripción                                              | Obligatorio |
|-----------|----------|----------------------------------------------------------|-------------|
| `id`      | `string` | Identificador único del pago.                            | Sí          |
| `estatus` | `string` | Nuevo estado del pago (`completado`, `fallido`).         | Sí          |

**Respuesta**:

| Campo                 | Tipo     | Descripción                                  |
|-----------------------|----------|----------------------------------------------|
| `id_pago`             | `string` | Identificador único del pago.                |
| `estatus`             | `string` | Nuevo estado del pago.                       |
| `fecha_actualizacion` | `string` | Fecha y hora de la actualización del estatus.|

**Ejemplo de Solicitud**:

```http
PUT /api/pagos/123456789/estatus
```

```json
{
  "estatus": "completado"
}
```

**Ejemplo de Respuesta**:

```json
{
  "id_pago": "123456789",
  "estatus": "completado",
  "fecha_actualizacion": "2024-09-03T13:45:00Z"
}
```

### 4. **Notificación de Cambio de Estatus (Automático)**

- **Descripción**: Este proceso se activa automáticamente cuando se cambia el estatus de un pago. Una notificación es enviada a RabbitMQ con los detalles del cambio.

**Contenido del Mensaje**:

| Campo           | Tipo     | Descripción                               |
|-----------------|----------|-------------------------------------------|
| `id_pago`       | `string` | Identificador único del pago.             |
| `estatus_nuevo` | `string` | Nuevo estado del pago.                    |
| `fecha_cambio`  | `string` | Fecha y hora en que se realizó el cambio. |

**Ejemplo de Mensaje**:

```json
{
  "id_pago": "123456789",
  "estatus_nuevo": "completado",
  "fecha_cambio": "2024-09-03T13:45:00Z"
}
```

## ✅ Conclusión

La documentación del API de gestión de pagos proporciona una guía clara y detallada sobre cómo interactuar con los diferentes endpoints del sistema. Cada sección incluye:

- **Descripción clara de los endpoints**: Detallando la funcionalidad y propósito de cada uno.
- **Especificación de los parámetros de entrada y salida**: Indicando los tipos de datos esperados y los campos que deben ser enviados y recibidos.
- **Ejemplos de uso**: Proporcionando ejemplos prácticos que ilustran cómo deben formarse las solicitudes y cómo se verán las respuestas, facilitando la integración y el uso del API.

---

# ⚠️ Lista de Riesgos del Proyecto

### 1. **Identificación de Riesgos Relevantes**

A continuación, se enumeran algunos de los riesgos más significativos que podrían afectar el éxito del proyecto de desarrollo del API de gestión de pagos:

- **Pérdida de datos al cambiar el estatus de un pago**: Existe el riesgo de que, durante el proceso de actualización del estatus de un pago, se produzca una pérdida de datos debido a errores en la transacción o fallos en la base de datos.
- **Fallas en la notificación a RabbitMQ**: Si RabbitMQ no recibe la notificación de cambio de estatus, otros sistemas dependientes pueden no reaccionar correctamente, causando inconsistencias en el flujo de trabajo.
- **Sobrecarga del sistema bajo alta demanda**: El API podría experimentar una sobrecarga bajo condiciones de alta demanda, lo que podría afectar la disponibilidad y el rendimiento del sistema.

### 2. **Evaluación del Impacto y Probabilidad**

Cada riesgo se evalúa en función de su impacto y probabilidad:

| Riesgo                                        | Impacto  | Probabilidad | Nivel de Riesgo |
|----------------------------------------------|----------|--------------|----------------|
| Pérdida de datos al cambiar el estatus        | Alto     | Medio        | Alto           |
| Fallas en la notificación a RabbitMQ          | Medio    | Alto         | Alto           |
| Sobrecarga del sistema bajo alta demanda      | Alto     | Medio        | Alto           |

### 3. **Propuesta de Estrategias de Mitigación Realistas**

Para cada riesgo identificado, se proponen las siguientes estrategias de mitigación:

- **Pérdida de datos al cambiar el estatus**:
  - Realizar copias de seguridad regulares de la base de datos para evitar la pérdida de información crítica.
  
- **Fallas en la notificación a RabbitMQ**:
  - Implementar un mecanismo de reintento automático para asegurar que los mensajes no entregados se vuelvan a intentar hasta que se confirme la entrega exitosa.
  - Monitorear RabbitMQ y establecer alertas para fallos en la entrega de mensajes.

- **Sobrecarga del sistema bajo alta demanda**:
  - Utilizar técnicas de caché para reducir la carga en la base de datos durante picos de tráfico.

---

Esta lista de riesgos proporciona una visión clara de los desafíos potenciales que podrían afectar el proyecto, junto con estrategias prácticas para mitigar dichos riesgos y asegurar el éxito del desarrollo e implementación del API. Con estas medidas, es posible reducir significativamente la probabilidad de problemas graves y mantener la integridad y seguridad del sistema.
