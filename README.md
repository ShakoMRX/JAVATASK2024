# Spring Boot Application with Docker Compose

## Overview
This project consists of two Spring Boot modules: `user-management-service` and `order-management-service`. The application is containerized using Docker Compose and includes services like PostgreSQL, Hazelcast, Zookeeper, Kafka, Kafdrop, and Kafka UI for a comprehensive development and monitoring environment.

## Prerequisites
- Docker
- Docker Compose

## Services
- **PostgreSQL**: Database service.
- **Hazelcast**: In-memory data grid for caching.
- **Zookeeper**: Kafka dependency for coordination.
- **Kafka**: Distributed streaming platform.
- **Kafdrop**: Web UI for viewing Kafka topics and messages.
- **Kafka UI**: Another web UI for Kafka management.
- **User Management Service**: Spring Boot microservice.
- **Order Management Service**: Spring Boot microservice.

## Getting Started

### Build and Run the Application
To build and run the application with Docker Compose, follow these steps:

1. **Clone the repository:**
    ```bash
    git clone <repository-url>
    cd <repository-directory>
    ```

2. **Build the Docker images:**
    ```bash
    docker-compose build
    ```

3. **Run the services:**
    ```bash
    docker-compose up
    ```

### Accessing Services

- **PostgreSQL**: Available at `localhost:5432`
- **Hazelcast**: Available at `localhost:5701`
- **Zookeeper**: Available at `localhost:2181`
- **Kafka**: Available at `localhost:9092` and `localhost:29092`
- **Kafdrop**: Access the Kafka web UI at [http://localhost:9000](http://localhost:9000)
- **Kafka UI**: Access the Kafka UI at [http://localhost:8085](http://localhost:8085)
- **User Management Service**: Access the service at [http://localhost:8081](http://localhost:8081)
- **Order Management Service**: Access the service at [http://localhost:8083](http://localhost:8083)

### Swagger UI
Swagger UI is available for both microservices to explore and test the APIs.

- **User Management Service** Swagger UI: [http://localhost:8081/swagger-ui.html](http://localhost:8081/swagger-ui.html)
- **Order Management Service** Swagger UI: [http://localhost:8083/swagger-ui.html](http://localhost:8083/swagger-ui.html)

## Docker Compose File
Below is the `docker-compose.yml` file used to define and configure the services:

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  hazelcast:
    image: hazelcast/hazelcast:latest
    ports:
      - "5701:5701"
    environment:
      - HZ_NETWORK_JOIN_MULTICAST_ENABLED=false
      - HZ_NETWORK_JOIN_TCPIP_ENABLED=true
      - HZ_NETWORK_JOIN_TCPIP_MEMBERS=hazelcast

  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT_INTERNAL://kafka:29092,PLAINTEXT_EXTERNAL://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT_INTERNAL:PLAINTEXT,PLAINTEXT_EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_CONFLUENT_SUPPORT_METRICS_ENABLE: 'false'
    ports:
      - "9092:9092"
      - "29092:29092"

  kafdrop:
    image: obsidiandynamics/kafdrop
    depends_on:
      - kafka
    environment:
      KAFKA_BROKERCONNECT: kafka:29092
    ports:
      - "9000:9000"

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    depends_on:
      - kafka
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:29092
      KAFKA_CLUSTERS_0_ZOOKEEPER: zookeeper:2181
    ports:
      - "8085:8080"

  user-management-service:
    build:
      context: ./user-management-service
      dockerfile: Dockerfile
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/mydb
      SPRING_DATASOURCE_USERNAME: user
      SPRING_DATASOURCE_PASSWORD: password
      SPRING_JPA_HIBERNATE_DDL_AUTO: update
      SPRING_CACHE_TYPE: hazelcast
      SPRING_HAZELCAST_CONFIG: classpath:hazelcast.xml
    depends_on:
      - postgres
      - hazelcast
      - kafka
    ports:
      - "8081:8081"

  order-management-service:
    build:
      context: ./order-management-service
      dockerfile: Dockerfile
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/mydb
      SPRING_DATASOURCE_USERNAME: user
      SPRING_DATASOURCE_PASSWORD: password
      SPRING_JPA_HIBERNATE_DDL_AUTO: update
      SPRING_CACHE_TYPE: hazelcast
      SPRING_HAZELCAST_CONFIG: classpath:hazelcast.xml
    depends_on:
      - postgres
      - hazelcast
      - kafka
    ports:
      - "8083:8083"

volumes:
  postgres_data:
