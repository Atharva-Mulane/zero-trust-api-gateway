# Zero-Trust API Gateway

A production-grade Zero-Trust API Gateway built with Spring Boot 3, Spring Cloud Gateway, Keycloak, Redis, and Zipkin — deployed on Railway with a React dashboard on Vercel.

## Live Demo
- **Dashboard:** https://zero-trust-gateway-dashboard.vercel.app
- **Gateway API:** https://zero-trust-api-gateway-production.up.railway.app/actuator/health

## Architecture

Client → Spring Cloud Gateway (Railway)
              ├── JWT Validation (Keycloak)
              ├── Rate Limiting (Redis)
              ├── Request Logging
              ├── Header Sanitization
              └── Distributed Tracing (Zipkin)
                        ↓
                 Mock Backend Service

## Tech Stack

| Component | Technology |
|-----------|------------|
| Gateway | Spring Cloud Gateway |
| Auth | Keycloak 24 + OAuth2 JWT |
| Rate Limiting | Redis (Reactive) |
| Tracing | Zipkin + Micrometer Brave |
| Monitoring | Spring Boot Actuator |
| Frontend | React + Vite |
| Deployment | Railway + Vercel |
| Containerization | Docker + Docker Compose |

## Features

- **Zero Trust Security** — Every request must carry a valid JWT token
- **JWT Validation** — Tokens validated against Keycloak's public keys
- **Rate Limiting** — 10 req/sec per user, burst of 20, tracked in Redis
- **Request Sanitization** — Strips Cookie headers, injects X-Gateway-Source
- **Distributed Tracing** — Full request traces sent to Zipkin
- **Health Monitoring** — Actuator endpoints for health, metrics, routes
- **React Dashboard** — Live UI for health, routes, metrics, and token testing

## Running Locally

### Prerequisites
- Java 21
- Docker Desktop
- Maven

### Start Infrastructure
```
docker compose up -d
```

### Run Gateway
``` 
./mvnw spring-boot:run
```

### Run Mock Backend
``` 
cd ../mock-service
mvn spring-boot:run -Dspring-boot.run.arguments=--server.port=8081
```

### Run Dashboard
``` 
cd ../../gateway-dashboard
npm run dev
```

### Setup Keycloak
1. Go to http://localhost:8080
2. Create realm: `zerotrust`
3. Create client: `gateway-client` (with client authentication)
4. Create user: `testuser` / `testpass`

### Get Token & Test
``` 
curl -s -X POST http://localhost:8080/realms/zerotrust/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password&client_id=gateway-client&client_secret=YOUR_SECRET&username=testuser&password=testpass"

curl -s http://localhost:9090/api/test -H "Authorization: Bearer YOUR_TOKEN"
```

## API Endpoints

| Endpoint | Access | Description |
|----------|--------|-------------|
| GET /actuator/health | Public | Gateway health |
| GET /actuator/gateway/routes | Public | Registered routes |
| GET /actuator/metrics | Public | Metrics |
| GET /api/** | JWT Required | Proxied to backend |

## Project Structure

```
api-gateway/
├── src/main/java/com/zerotrust/gateway/
│   ├── ApiGatewayApplication.java
│   ├── config/
│   │   ├── SecurityConfig.java       # JWT + CORS
│   │   ├── GatewayConfig.java        # Routes + Rate Limiting
│   │   └── RateLimiterConfig.java    # Redis Key Resolver
│   └── filter/
│       └── LoggingFilter.java        # Request Logging
├── src/main/resources/
│   └── application.properties
├── docker-compose.yml
└── Dockerfile

## Author
Atharva Mulane
