# Demo Guide

A concise walkthrough to demo each microservice individually on macOS.

## Prerequisites
- Java 21+ and Maven (or ./mvnw) installed and on PATH.
- Node.js 12+ with npm.
- Python available for node-gyp (default on macOS).
- Ports free: 8080, 8090, 8100, 8110, 3000, 3010, 3020, 50051, 61613, 61616, 9000.

## Common notes
- Each Spring Boot service uses file-backed H2 with user `sa` / password `sa` and recreates schema on start (`create-drop`).
- Start backends before frontends. Use separate terminals. Stop with Ctrl+C.
- Health endpoints: `/actuator/health`. H2 consoles at `/console` per backend.

## Service-by-service run instructions

### Customer Core (customer-core)
Purpose: master customer data service; other backends consume it.
Run:
```
cd customer-core
mvn spring-boot:run
```
Access:
- API base: http://localhost:8110
- Health: http://localhost:8110/actuator/health
- H2 console: http://localhost:8110/console (JDBC `jdbc:h2:file:./customercore`, user `sa`, pass `sa`).

### Customer Management Backend (customer-management-backend)
Purpose: supports customer-service agents (lookup, interactions); depends on Customer Core.
Run:
```
cd customer-management-backend
mvn spring-boot:run
```
Access:
- API base: http://localhost:8100
- Health: http://localhost:8100/actuator/health
- H2 console: http://localhost:8100/console (JDBC `jdbc:h2:file:./customermanagement`).
- Config uses `customercore.baseURL=http://localhost:8110`.

### Policy Management Backend (policy-management-backend)
Purpose: policies API; emits policy events over ActiveMQ for risk service.
Run:
```
cd policy-management-backend
mvn spring-boot:run
```
Access:
- API base: http://localhost:8090
- Health: http://localhost:8090/actuator/health
- H2 console: http://localhost:8090/console (JDBC `jdbc:h2:file:./policymanagement`).
- ActiveMQ brokers: STOMP 61613, TCP 61616. Depends on Customer Core via `customercore.baseURL`.

### Customer Self-Service Backend (customer-self-service-backend)
Purpose: serves the self-service UI (registration, address change); depends on Customer Core and Policy events.
Run:
```
cd customer-self-service-backend
mvn spring-boot:run
```
Access:
- API base: http://localhost:8080
- Health: http://localhost:8080/actuator/health
- H2 console: http://localhost:8080/console (JDBC `jdbc:h2:file:./customerselfservice`).
- Connects to Customer Core (`customercore.baseURL`) and Policy broker (`policymanagement.tcpBrokerBindAddress=tcp://localhost:61616`).

### Spring Boot Admin (spring-boot-admin)
Purpose: monitors the Spring Boot apps.
Run:
```
cd spring-boot-admin
mvn spring-boot:run
```
Access: http://localhost:9000 (apps self-register if `spring.boot.admin.client.url` is set).

### Customer Self-Service Frontend (customer-self-service-frontend)
Purpose: React UI for customers.
Run:
```
cd customer-self-service-frontend
npm install   # first time
npm start
```
Access: http://localhost:3000 (talks to 8080/8090/8100 via env defaults).

### Customer Management Frontend (customer-management-frontend)
Purpose: React UI for agents (lookup, chat).
Run:
```
cd customer-management-frontend
npm install   # first time
npm start
```
Access: http://localhost:3020 (talks to 8100 and others via env defaults).

### Policy Management Frontend (policy-management-frontend)
Purpose: Vue UI for policy staff.
Run:
```
cd policy-management-frontend
npm install   # first time
npm run serve
```
Access: http://localhost:3010 (talks to 8090).

### Risk Management Server (risk-management-server) — optional
Purpose: Node service consuming policy events via ActiveMQ; exposes gRPC (50051) for reports.
Run:
```
cd risk-management-server
npm install   # first time
npm start
```
Note: expects ActiveMQ at policy-management-backend (61613/61616) and events on `newpolicies` queue.

### Risk Management Client (risk-management-client) — optional
Purpose: CLI to request reports from risk server.
Run:
```
cd risk-management-client
npm install   # first time
npm start
```
Configure target host/port in its config if needed.

## Suggested live demo flow
1) Start backends: customer-core → policy-management-backend → customer-management-backend → customer-self-service-backend → spring-boot-admin.
2) Start UIs: customer-self-service-frontend (3000), customer-management-frontend (3020), policy-management-frontend (3010).
3) Show customer journey: in 3000 view policy/address; in 3020 look up same customer and show chat UI; in 3010 show policies list.
4) Show monitoring: open http://localhost:9000 and confirm services are registered.
5) Show data isolation: open each service’s `/console` to display its own tables.
6) (Optional) Start risk-management-server and note policy events flowing from 8090 to 50051 via ActiveMQ.

## Troubleshooting quick hits
- Port in use: stop the conflicting process or change `server.port` in that service’s properties.
- Empty tables: hit an API or UI action first so data is created, then refresh H2 console.
- ActiveMQ warnings on policy backend: expected until broker binds; they clear once running.
- Slow first start: Maven downloads and npm install are one-time per service.
