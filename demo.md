# Demo Guide (Serial Startup)

## Prereqs
- Java 17+, Maven 3.9+
- Node.js 18+ and npm
- Docker (for ActiveMQ via docker-compose if you prefer containerized broker)

## 1) Start Infrastructure
- (Option A) Run embedded ActiveMQ configs already in services.
- (Option B) Containerized broker (recommended):
  ```bash
  docker compose up -d activemq
  ```
  Default console: http://localhost:8161 (admin/admin).

## 2) Start Spring Boot Admin (port 9000)
```bash
cd spring-boot-admin
mvn spring-boot:run
```
Keep it running so other services can register without warnings.

## 3) Start Core Backends (in this order)
Open separate terminals:

1) Customer Core (8110)
```bash
cd customer-core
mvn spring-boot:run
```

2) Policy Management Backend (8090)
```bash
cd policy-management-backend
mvn spring-boot:run
```

3) Customer Management Backend (8100)
```bash
cd customer-management-backend
mvn spring-boot:run
```

4) Customer Self-Service Backend (8080)
```bash
cd customer-self-service-backend
mvn spring-boot:run
```

(Optionally) Risk Management Server (gRPC, 50051)
```bash
cd risk-management-server
npm install
npm start
```

## 4) Start Frontends
Open separate terminals:

- Policy Management Frontend (3010)
```bash
cd policy-management-frontend
npm install
npm start
```

- Customer Management Frontend (3020)
```bash
cd customer-management-frontend
npm install
npm start
```

- Customer Self-Service Frontend (3000)
```bash
cd customer-self-service-frontend
npm install
npm start
```

## 5) Validate Health
- Spring Boot Admin: http://localhost:9000 (all services should show “UP”)
- Customer Core actuator: http://localhost:8110/actuator/health
- Policy Management actuator: http://localhost:8090/actuator/health
- Customer Management actuator: http://localhost:8100/actuator/health
- Self-Service actuator: http://localhost:8080/actuator/health
- ActiveMQ console (if using container): http://localhost:8161

## 6) Demo Flows (suggested)
- Login to Customer Self-Service (3000) → View policies → Update address (triggers event to Policy Management via ActiveMQ).
- Admin flow on Policy Management UI (3010) → Review updated policy/risk after address change.
- Backoffice flow on Customer Management UI (3020) → Verify customer profile reflects updates.

## 7) Shutdown (reverse order)
- Stop frontends (Ctrl+C each).
- Stop backends (Ctrl+C each).
- Stop SBA (Ctrl+C).
- If using Dockerized ActiveMQ: `docker compose down`.