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

## 🚀 How to Run

### Prerequisites

- Java 21 or higher
- Maven 3.8+

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

### Running with Docker (Coming Soon)

```bash
docker run -p 8761:8761 ghcr.io/danlearnings/ecommerce-service-registry:latest
```

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

## 🔧 Integration with Other Services

Other services connect to this registry by adding to their `application.yml`:

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    fetch-registry: true
    register-with-eureka: true
```

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

## 📚 Additional Resources

- [Spring Cloud Netflix Documentation](https://spring.io/projects/spring-cloud-netflix)
- [Netflix Eureka Wiki](https://github.com/Netflix/eureka/wiki)
- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)

## 🔗 Related Services

- [Config Server](https://github.com/DanLearnings/ecommerce-config-server) - Centralized configuration (registers with this service)
- [API Gateway](https://github.com/DanLearnings/ecommerce-api-gateway) - Routes requests to services discovered here
- [Inventory Service](https://github.com/DanLearnings/ecommerce-inventory-service) - Business service that registers here

## 👨‍💻 Maintainer

**Dan Learnings**
- GitHub: [@DanrleyBrasil](https://github.com/DanrleyBrasil)
- Organization: [DanLearnings](https://github.com/DanLearnings)

---

**Last Updated:** October 29, 2025
