# Service Registry (Eureka Server)

## 🎯 Purpose

The Service Registry is the heart of the microservices ecosystem. It acts as a **service discovery server** using Netflix Eureka, allowing all microservices to register themselves and discover other services without hardcoded URLs.

## 🔌 Port

- **Default Port:** `8761`
- **Dashboard URL:** http://localhost:8761

## 🛠️ Tech Stack

- **Framework:** Spring Boot 3.5.7
- **Service Discovery:** Spring Cloud Netflix Eureka Server
- **Monitoring:** Spring Boot Actuator
- **Containerization:** Docker
- **Build Tool:** Maven

## 📦 Dependencies

This service has **no external dependencies** - it should be the **first service** to start.

## ⚙️ Configuration

### Key Configuration (`application.yml`):

```yaml
server:
  port: 8761

spring:
  application:
    name: eureka-server

eureka:
  client:
    register-with-eureka: false    # Don't register with itself
    fetch-registry: false           # Don't fetch registry from itself
    service-url:
      defaultZone: http://localhost:8761/eureka/
  server:
    enable-self-preservation: false # Disable for development
```

### Important Notes:

- `register-with-eureka: false` - Since this IS the registry, it doesn't need to register with itself
- `fetch-registry: false` - No need to fetch the registry from itself
- `enable-self-preservation: false` - Disabled for development to immediately remove failed instances

---

## 🚀 How to Run

### Prerequisites

- Java 21 or higher
- Maven 3.8+
- Docker (for containerized deployment)

---

## 🐳 Option 1: Running with Docker (Recommended)

### Quick Start

```bash
# Run the pre-built image
docker run -d \
  --name service-registry \
  -p 8761:8761 \
  ecommerce-service-registry:latest
```

### Building the Docker Image

```bash
# Clone the repository
git clone https://github.com/DanLearnings/ecommerce-service-registry.git
cd ecommerce-service-registry

# Build the Docker image
docker build -t ecommerce-service-registry:latest .

# Run the container
docker run -d \
  --name service-registry \
  -p 8761:8761 \
  ecommerce-service-registry:latest
```

### Running in Docker Network (for multi-container setup)

```bash
# Create network (if not exists)
docker network create ecommerce-network

# Run container in network
docker run -d \
  --name service-registry \
  --network ecommerce-network \
  -p 8761:8761 \
  ecommerce-service-registry:latest
```

### Docker Environment Variables

```bash
# Run with custom memory settings
docker run -d \
  --name service-registry \
  -p 8761:8761 \
  -e JAVA_OPTS="-Xmx1g -Xms512m" \
  ecommerce-service-registry:latest
```

### Viewing Logs

```bash
# View logs
docker logs service-registry

# Follow logs in real-time
docker logs -f service-registry
```

### Stopping and Removing

```bash
# Stop the container
docker stop service-registry

# Remove the container
docker rm service-registry

# Stop and remove in one command
docker rm -f service-registry
```

---

## 💻 Option 2: Running with Maven (Development)

### Running Locally

```bash
# Clone the repository
git clone https://github.com/DanLearnings/ecommerce-service-registry.git
cd ecommerce-service-registry

# Run with Maven
mvn spring-boot:run

# Or build and run the JAR
mvn clean package
java -jar target/ecommerce-service-registry-1.0.0.jar
```

---

## 🔍 Endpoints

### Eureka Dashboard
- **URL:** http://localhost:8761
- **Description:** Web UI showing all registered services

### Health Check
- **URL:** http://localhost:8761/actuator/health
- **Expected Response:**
```json
{
  "status": "UP"
}
```

### Eureka API
- **URL:** http://localhost:8761/eureka/apps
- **Description:** REST API to view registered applications (XML format)

## ✅ Health Check

Verify the service is running correctly:

```bash
# Check if service is up
curl http://localhost:8761/actuator/health

# View the dashboard in browser
open http://localhost:8761
```

**Expected behavior:**
- The dashboard should load showing "Instances currently registered with Eureka"
- Initially, the instances list will be empty
- As other services start, they will appear in this dashboard

---

## 🔧 Integration with Other Services

Other services connect to this registry by adding to their `application.yml`:

### When running locally (Maven):
```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    fetch-registry: true
    register-with-eureka: true
```

### When running in Docker:
```yaml
eureka:
  client:
    service-url:
      defaultZone: http://service-registry:8761/eureka/  # Use container name
    fetch-registry: true
    register-with-eureka: true
```

---

## 🐛 Troubleshooting

### Port 8761 already in use

**Symptom:** Service fails to start with "Port 8761 was already in use"

**Solution:**
```bash
# Windows
netstat -ano | findstr :8761
taskkill /PID <PID> /F

# Linux/Mac
lsof -i :8761
kill -9 <PID>

# Docker: Use different host port
docker run -p 9761:8761 ecommerce-service-registry:latest
```

### Dashboard shows "No instances available"

**Symptom:** Eureka dashboard loads but shows no registered services

**Solution:**
- This is normal if no other services are running
- Start other services (Config Server, API Gateway, etc.) and they will appear
- Wait 30 seconds for heartbeat registration

### Service keeps de-registering and re-registering

**Symptom:** Services appear and disappear from the dashboard

**Solution:**
- Check network connectivity between services
- Ensure services have proper health checks configured
- Verify `eureka.instance.lease-renewal-interval-in-seconds` is appropriate for your environment

### Docker: Container exits immediately

**Symptom:** Container starts but exits right away

**Solution:**
```bash
# Check logs for errors
docker logs service-registry

# Common issues:
# - Port already in use (change host port mapping)
# - Insufficient memory (adjust JAVA_OPTS)
# - Configuration errors (check application.yml)
```

### Docker: Cannot access dashboard from host

**Symptom:** Container is running but cannot access http://localhost:8761

**Solution:**
```bash
# Verify port mapping
docker ps | grep service-registry

# Ensure you're using correct port mapping
docker run -p 8761:8761 ecommerce-service-registry:latest

# Check if container is healthy
docker inspect service-registry | grep Health
```

---

## 🐳 Docker Image Details

### Multi-stage Build

The Dockerfile uses a multi-stage build:
- **Stage 1 (build):** Uses Maven image to compile the application
- **Stage 2 (runtime):** Uses lightweight JRE image to run the application

### Image Size Optimization

- Base image: `eclipse-temurin:21-jre-alpine`
- Approximate size: ~200MB
- Non-root user for security
- Health check included

### Health Check

The Docker image includes a built-in health check:
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8761/actuator/health || exit 1
```

---

## 📚 Additional Resources

- [Spring Cloud Netflix Documentation](https://spring.io/projects/spring-cloud-netflix)
- [Netflix Eureka Wiki](https://github.com/Netflix/eureka/wiki)
- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
- [Docker Documentation](https://docs.docker.com/)

## 🔗 Related Services

- [Config Server](https://github.com/DanLearnings/ecommerce-config-server) - Centralized configuration (registers with this service)
- [API Gateway](https://github.com/DanLearnings/ecommerce-api-gateway) - Routes requests to services discovered here
- [Inventory Service](https://github.com/DanLearnings/ecommerce-inventory-service) - Business service that registers here

---

## 👨‍💻 Maintainer

**Danrley Brasil (Dan Learnings)**
- Personal GitHub: [@DanrleyBrasil](https://github.com/DanrleyBrasil)
- Organization: [DanLearnings](https://github.com/DanLearnings)

---

**Last Updated:** October 29, 2025
