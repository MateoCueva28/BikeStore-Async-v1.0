# 🚴 BikeStore Async v1.0

**Realizado por: Mateo Cueva**

**Asignatura: Diseño y Arquitectura de Software**

> **Arquitectura Asincrónica de Microservicios con Spring Boot y RabbitMQ**

---

##  Descripción General

**BikeStore Async v1.0** es una aplicación académica que demuestra cómo implementar una **arquitectura asincrónica basada en eventos** utilizando **RabbitMQ** como broker de mensajes.

El proyecto simula un **e-commerce de bicicletas** donde:
1. Un cliente confirma su orden mediante una API REST
2. La orden se procesa de forma **asincrónica** a través de múltiples servicios desacoplados
3. Se gestiona el **procesamiento de pagos** con reintentos automáticos
4. Se envían **confirmaciones por email** de forma independiente
5. Los **fallos se capturan** en una Dead-Letter Queue para análisis offline



##  Características

### Core Features
- ✅ **API REST** para crear órdenes (`POST /api/orders/create`)
- ✅ **Publicador de mensajes** (OrderProducer) que publica órdenes en RabbitMQ
- ✅ **Procesador de pagos** (PaymentWorker) con lógica de reintentos (máx 3 intentos)
- ✅ **Gestor de emails** (EmailWorker) que envía confirmaciones solo si pago fue exitoso
- ✅ **Dead-Letter Queue (DLQ)** para órdenes que fallan permanentemente

### Características Técnicas
- ✅ **Simulación realista:** 50% de probabilidad de fallo en pagos
- ✅ **Logging detallado:** Cada evento registra `orderId`, `timestamp`, `thread`, `estado`
- ✅ **Manejo de errores:** Reintentos automáticos, circuito breaker listo
- ✅ **Message Converter:** Serialización/deserialización segura con Jackson
- ✅ **Health checks:** Endpoints para verificar estado del servicio

---

### Componentes

| Componente | Responsabilidad | Tecnología |
|-----------|-----------------|-----------|
| **OrderProducer** | API REST que recibe órdenes | Spring Web, REST |
| **PaymentWorker** | Procesa pagos con reintentos | Spring AMQP, RabbitListener |
| **EmailWorker** | Envía confirmaciones | Spring AMQP, RabbitListener |
| **RabbitMQ** | Broker de mensajes | Docker, RabbitMQ Management |


---

## 🛠️ Stack Tecnológico

| Tecnología | Versión | Propósito |
|-----------|---------|----------|
| **Java** | 23      | Lenguaje base |
| **Spring Boot** | 3.1.5+  | Framework |
| **Spring AMQP** | Latest  | Cliente RabbitMQ |
| **RabbitMQ** | 3.12    | Broker de mensajes |
| **Docker** | Latest  | Contenedorización de RabbitMQ |
| **Maven** | 3.9+    | Gestor de dependencias |
| **Lombok** | Latest  | Reducción de boilerplate |
| **Jackson** | Latest  | Serialización JSON |


---

##  Instalación

### Requisitos previos

```bash
# Java 21+ (verificar)
java -version

# Docker y Docker Compose (verificar)
docker --version
docker-compose --version

# Maven (verificar)
mvn -version

```

Si te falta algo, descarga desde:
- **Java 21:** https://www.oracle.com/java/technologies/downloads/
- **Docker Desktop:** https://www.docker.com/products/docker-desktop
- **Maven:** https://maven.apache.org/download.cgi

### Pasos de instalación

#### 1. Clonar el repositorio

```bash
git clone https://github.com/MateoCueva28/BikeStore-Async-v1.0.git
cd BikeStore-Async-v1.0
```

#### 2. Verificar estructura del proyecto

```bash
.
├── docker-compose.yml          # Configuración RabbitMQ
├── pom.xml                     # Dependencias Maven
├── src/
│   └── main/
│       ├── java/com/bikestore/
│       │   ├── BikeStoreAsyncApplication.java
│       │   ├── config/
│       │   │   └── RabbitMQConfig.java
│       │   ├── model/
│       │   │   └── Order.java
│       │   ├── service/
│       │   │   ├── OrderProducer.java
│       │   │   ├── PaymentWorker.java
│       │   │   └── EmailWorker.java
│       │   └── controller/
│       │       └── OrderController.java
│       └── resources/
│           └── application.yml
└── README.md
```

#### 3. Levantar RabbitMQ con Docker

```bash
# Crear y ejecutar el contenedor
docker-compose up -d

# Verificar que está corriendo
docker-compose ps

# Debería mostrar:
# NAME                    STATUS
# bikestore-rabbitmq      Up (healthy)
```

#### 4. Compilar el proyecto

```bash
# Opción A: Con Maven instalado
mvn clean compile


# Opción B: Con Maven Wrapper (recomendado)
./mvnw clean compile
```
---

## 🚀 Ejecución

### Iniciar la aplicación

```bash
mvnw spring-boot:run
```

### Verificar que está corriendo

```bash
# En otra terminal/CMD, verifica health check
curl http://localhost:8080/api/orders/health

# Respuesta esperada:
# {
#   "status": "UP",
#   "service": "BikeStore Order Service"
# }
```

### Acceder a la consola de RabbitMQ

Abre tu navegador:

```
URL: http://localhost:15672
Usuario: admin
Contraseña: admin123
```

Aquí verás:
- **Connections:** Conexiones activas
- **Channels:** Canales de comunicación
- **Queues:** Colas (payment.queue, email.queue, dlq.queue)
- **Message rates:** Mensajes procesados por segundo

---

## 💻 Uso

### Crear una orden (Flujo completo)

#### Opción 1: Con cURL

```bash
curl -X POST http://localhost:8080/api/orders/create \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "CUST-001",
    "customerName": "Juan Pérez",
    "customerEmail": "juan.perez@example.com",
    "bikeName": "Mountain Bike Pro",
    "quantity": 1,
    "totalAmount": 1500.00,
    "cardToken": "tok_visa_4242"
  }'
```

#### Opción 2: Con Postman (Usada por mi)

1. Crear nuevo request → **POST**
2. URL: `http://localhost:8080/api/orders/create`
3. Headers: `Content-Type: application/json`
4. Body (raw JSON):

```json
{
  "customerId": "CUST-001",
  "customerName": "Juan Pérez",
  "customerEmail": "juan.perez@example.com",
  "bikeName": "Mountain Bike Pro",
  "quantity": 1,
  "totalAmount": 1500.00,
  "cardToken": "tok_visa_4242"
}
```

5. Presionar en **Send**

#### Respuesta esperada

```json
{
  "success": true,
  "message": "Orden recibida. Se procesará en breve.",
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "status": "PENDING"
}
```

El cliente recibe esta respuesta **inmediatamente** (202 ACCEPTED), mientras el procesamiento continúa en background.

### Ver logs en consola

En la terminal donde ejecutaste la app, verás:

```
═══════════════════════════════════════════════════════════════
[ORDER PRODUCER] Publicando orden
  ├─ Orden ID: 550e8400-e29b-41d4-a716-446655440000
  ├─ Cliente: Juan Pérez (juan.perez@example.com)
  ├─ Bicicleta: Mountain Bike Pro x1
  ├─ Monto: $1500.00
  ├─ Timestamp: 2024-05-09T14:23:45.123456
  └─ Thread: http-nio-8080-exec-1
═══════════════════════════════════════════════════════════════

───────────────────────────────────────────────────────────────
[PAYMENT WORKER] Recibida orden para procesar pago
  ├─ Orden ID: 550e8400-e29b-41d4-a716-446655440000
  ├─ Monto: $1500.00
  ├─ Timestamp: 1715338425234
  └─ Thread: org.springframework.amqp.rabbit.RabbitListenerEndpointContainer#0-1
───────────────────────────────────────────────────────────────

[PAYMENT WORKER] PAGO EXITOSO
  ├─ Orden ID: 550e8400-e29b-41d4-a716-446655440000
  ├─ Estado: PAID
  └─ Thread: org.springframework.amqp.rabbit.RabbitListenerEndpointContainer#0-1

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[EMAIL WORKER] Procesando solicitud de email
  ├─ Orden ID: 550e8400-e29b-41d4-a716-446655440000
  ├─ Cliente: Juan Pérez (juan.perez@example.com)
  ├─ Estado de pago: PAID
  ├─ Timestamp: 1715338426042
  └─ Thread: org.springframework.amqp.rabbit.RabbitListenerEndpointContainer#0-2
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✉[EMAIL WORKER] EMAIL ENVIADO EXITOSAMENTE
  ├─ Orden ID: 550e8400-e29b-41d4-a716-446655440000
  ├─ Destinatario: juan.perez@example.com
  ├─ Asunto: Pedido #550e8400... - Confirmación de compra
  ├─ Contenido: Gracias por comprar Mountain Bike Pro x1
  └─ Thread: org.springframework.amqp.rabbit.RabbitListenerEndpointContainer#0-2
```

### Casos de prueba recomendados

Ejecuta 5-6 órdenes seguidas para ver diferentes escenarios:

```bash
# Orden 1: Probablemente éxito
curl -X POST http://localhost:8080/api/orders/create \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "CUST-001",
    "customerName": "Alice Johnson",
    "customerEmail": "alice@example.com",
    "bikeName": "Road Bike Carbon",
    "quantity": 2,
    "totalAmount": 3200.00,
    "cardToken": "tok_4242"
  }'

# Orden 2: Probablemente éxito
curl -X POST http://localhost:8080/api/orders/create \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "CUST-002",
    "customerName": "Bob Smith",
    "customerEmail": "bob@example.com",
    "bikeName": "BMX Pro",
    "quantity": 1,
    "totalAmount": 800.00,
    "cardToken": "tok_5555"
  }'

# Orden 3: Probablemente falla (reintentos → DLQ)
curl -X POST http://localhost:8080/api/orders/create \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "CUST-003",
    "customerName": "Carol Davis",
    "customerEmail": "carol@example.com",
    "bikeName": "Electric Bike",
    "quantity": 1,
    "totalAmount": 2500.00,
    "cardToken": "tok_visa_123"
  }'

# Orden 4, 5, 6... (varía el resultado)
```

---

## 📨 Flujo de Mensajería

### Diagrama detallado de intercambio de mensajes

```
PASO 1: Cliente envía orden (HTTP)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
POST /api/orders/create
{
  "customerId": "CUST-001",
  "customerName": "Juan",
  "customerEmail": "juan@example.com",
  "bikeName": "Mountain Bike",
  "quantity": 1,
  "totalAmount": 1500.00,
  "cardToken": "tok_4242"
}
      ↓
Respuesta inmediata (202 ACCEPTED):
{
  "success": true,
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "status": "PENDING"
}

PASO 2: OrderProducer publica en RabbitMQ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Exchange: payment.exchange
Routing Key: payment.process
Mensaje:
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "customerId": "CUST-001",
  "customerName": "Juan",
  "customerEmail": "juan@example.com",
  "bikeName": "Mountain Bike",
  "quantity": 1,
  "totalAmount": 1500.00,
  "cardToken": "tok_4242",
  "status": "PENDING",
  "orderDate": "2024-05-09T14:23:45.123456"
}

PASO 3: PaymentWorker consume de payment.queue
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Recibe el mensaje → Simula pago (50% éxito)

ESCENARIO A: PAGO EXITOSO (50%)
┌─────────────────────────────────────────┐
│ PaymentWorker procesa exitosamente      │
│ Status = "PAID"                         │
│                                         │
│ Publica en: email.exchange              │
│ Routing Key: email.send                 │
│ Mensaje: Order (status=PAID)            │
└──────────┬──────────────────────────────┘
           ↓
    ┌──────────────────────┐
    │ EmailWorker consume  │
    │ email.queue          │
    │                      │
    │ Valida: status=PAID  │
    │ ✓ Envía email        │
    │                      │
    │ Log:                 │
    │ ✉️ EMAIL ENVIADO     │
    └──────────────────────┘

ESCENARIO B: PAGO FALLIDO (50%)
┌─────────────────────────────────────────┐
│ PaymentWorker intenta procesar pago     │
│ ✗ FALLO                                 │
│                                         │
│ Reintento 1 → ✗ FALLO                  │
│ Republica en: payment.exchange          │
│ Header: x-retry-count = 1               │
└──────────┬──────────────────────────────┘
           ↓ (con delay)
    ┌──────────────────────┐
    │ Reintento 1          │
    │ ✗ FALLO              │
    │ Retry count = 2      │
    └──────────┬───────────┘
               ↓
    ┌──────────────────────┐
    │ Reintento 2          │
    │ ✗ FALLO              │
    │ Retry count = 3      │
    └──────────┬───────────┘
               ↓
    ┌──────────────────────────────────────┐
    │ ✗ Máximo de reintentos alcanzado    │
    │                                      │
    │ Publica en: dlq.exchange             │
    │ Routing Key: dlq.payment.failed      │
    │ Mensaje: Order (status=FAILED)       │
    │                                      │
    │ Log:                                 │
    │ ❌ PAGO FALLIDO DESPUÉS DE 3 INTENTOS│
    │    → Dead-Letter Queue para análisis │
    └──────────────────────────────────────┘
```

### Configuración de Exchanges y Queues

```bash
# Exchanges (Topic type)
payment.exchange      # Enruta órdenes para procesar pago
email.exchange        # Enruta notificaciones de email
dlq.exchange          # Captura mensajes fallidos

# Queues
payment.queue         # Contiene órdenes a procesar
email.queue           # Contiene confirmaciones a enviar
dlq.queue             # Contiene órdenes fallidas

# Bindings
payment.queue    ← payment.exchange + "payment.process"
email.queue      ← email.exchange + "email.send"
dlq.queue        ← dlq.exchange + "dlq.*"

# Consumidores
PaymentWorker listens on: payment.queue
EmailWorker listens on: email.queue
(DLQ se consulta manualmente vía Management UI)
```
---

## 📊 Monitoreo

### RabbitMQ Management Console

Accede a: **http://localhost:15672** (admin/admin123)

**Qué observar:**
1. **Queues:** Número de mensajes pendientes
2. **Message rates:** Mensajes/segundo entrando y saliendo
3. **Consumers:** Cuántos consumidores están activos
4. **DLQ:** Mensajes fallidos (debería estar vacío en ejecución normal)

### Logs de la aplicación

Los logs en consola te muestran todo lo que sucede:
- Orden recibida
- Intento de pago
- Resultado (éxito/fallo)
- Email enviado o no
- Envío a DLQ si falló

**Busca estos patrones en los logs:**
```
[ORDER PRODUCER]      → Orden publicada
[PAYMENT WORKER]      → Pago procesando
PAGO EXITOSO       → Pago aprobado
  PAGO FALLIDO      → Fallo, reintentando
 FRACASO DEFINITIVO → Máximo de reintentos
[EMAIL WORKER]        → Email procesando
 EMAIL ENVIADO     → Confirmación enviada
```