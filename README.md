# Cocina Service — Arepa Express

Microservicio consumidor que recibe eventos de pedidos desde RabbitMQ, simula la preparación de las arepas y registra el avance en sus logs.

## Tecnologías

- Java 21
- Spring Boot 3.5.0
- Spring AMQP
- RabbitMQ
- Maven
- Docker

## Requisitos

- JDK 21
- Maven 3.9 o compatible
- RabbitMQ disponible
- Docker Desktop (opcional, para ejecutar el contenedor)

## Configuración de RabbitMQ

La aplicación lee estas variables de entorno:

| Variable | Valor predeterminado | Descripción |
|---|---|---|
| `RABBIT_HOST` | `localhost` | Host del servidor RabbitMQ |
| `RABBIT_USER` | `arepas` | Usuario de RabbitMQ |
| `RABBIT_PASS` | `arepas123` | Contraseña de RabbitMQ |

El puerto AMQP utilizado es `5672`.

## Cómo funciona

1. Declara el exchange topic `arepas.eventos`.
2. Declara la cola durable `cocina.pedidos` y la enlaza con la clave `pedido.*`.
3. Escucha los eventos de la cola, registra que comenzó la preparación, espera tres segundos y registra que el pedido quedó listo.

El evento recibido contiene `pedidoId`, `cliente`, `arepas` y `fecha`.

## Ejecutar localmente

Con RabbitMQ disponible, desde la carpeta `cocina-service` inicia el consumidor:

```bash
mvn spring-boot:run
```

Cuando `pedidos-service` publique un pedido en RabbitMQ, los mensajes de preparación y finalización aparecerán en la salida de la aplicación.

## Compilar

```bash
mvn -q package -DskipTests
```

El archivo JAR se genera dentro de `target/`.

## Construir y ejecutar con Docker

Primero compila el JAR y, desde esta carpeta, construye la imagen:

```bash
mvn -q package -DskipTests
docker build -t cocina-service .
docker run --rm \
  -e RABBIT_HOST=host.docker.internal \
  -e RABBIT_USER=arepas \
  -e RABBIT_PASS=arepas123 \
  cocina-service
```

Para ejecutar el contenedor en este laboratorio, configura `RABBIT_HOST` con la dirección del RabbitMQ desplegado en Azure y usa las credenciales definidas para ese broker.

## Estructura principal

```text
cocina-service/
├── src/main/java/com/arepa/cocina/
│   ├── domain/model/PedidoCreadoEvent.java
│   └── infrastructure/
│       ├── adapter/in/messaging/CocinaListener.java
│       └── config/RabbitConfig.java
├── src/main/resources/application.properties
├── .dockerignore
├── .gitignore
├── Dockerfile
└── pom.xml
```
