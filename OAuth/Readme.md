# Spring Boot Microservices with OAuth2 and Keycloak

## Overview
This project demonstrates secure communication between two Spring Boot microservices using OAuth2 client credentials flow and Keycloak.

- **Service A**: Client that requests a token using `client_credentials` and calls Service B
- **Service B**: Resource server with protected endpoint `/api/data`
- **Keycloak**: OAuth2 Authorization Server

---

## 📁 Project Structure
- `service-a/`: OAuth2 client microservice
- `service-b/`: OAuth2 resource server
- `keycloak-config/`: Contains `realm-export.json` to pre-configure Keycloak
- `docker-compose.yml`: Spins up Keycloak instance on port 8089

---

## 💻 Prerequisites
- Java 17+
- Maven 3.x
- Docker & Docker Compose

---

## 🔐 Keycloak Setup
Run the following to start Keycloak with pre-configured realm and clients:

```bash
docker-compose up -d
```

### Access Keycloak UI
- URL: http://localhost:8089
- Admin Username: `admin`
- Admin Password: `admin`

### Preconfigured:
- Realm: `myrealm`
- Client `service-a`: confidential, `client_credentials`
- Client `service-b`: bearer-only

---

## 🚀 Services

### Service A (port 8080)
- Calls Service B using token
- Endpoint: `GET /call-service-b`

```yaml
# application.yml
server:
  port: 8080

keycloak:
  token-uri: http://localhost:8089/realms/myrealm/protocol/openid-connect/token
  client-id: service-a
  client-secret: service-a-secret

service-b:
  url: http://localhost:8081
```

### Service B (port 8081)
- Protected endpoint: `GET /api/data`

```yaml
# application.yml
server:
  port: 8081
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8089/realms/myrealm
```

---

## ✅ How to Run
1. Start Keycloak:
```bash
docker-compose up -d
```

2. Build and run Service B:
```bash
cd service-b
./mvnw spring-boot:run
```

3. Build and run Service A:
```bash
cd service-a
./mvnw spring-boot:run
```

4. Test Service A calling Service B:
```bash
curl http://localhost:8080/call-service-b
```
Expected Output:
```
Protected data from Service B. Client ID: service-a
```

---

## 🔧 Technologies
- Spring Boot 3
- WebFlux
- OAuth2 Client / Resource Server
- Keycloak 24

---

## 🧪 Test Tokens Manually (Optional)
```bash
curl -X POST \
  -d "client_id=service-a" \
  -d "client_secret=service-a-secret" \
  -d "grant_type=client_credentials" \
  http://localhost:8089/realms/myrealm/protocol/openid-connect/token
```
Then use the token with:
```bash
curl -H "Authorization: Bearer <TOKEN>" http://localhost:8081/api/data
```

---

## 🛠 Developer Tips
- You can import the realm manually via the Keycloak Admin UI if auto-import fails.
- If Keycloak fails to start, run: `docker-compose down -v && docker-compose up --build`

---

## ❗ Troubleshooting
- **401 Unauthorized** from Service B:
  - Ensure Keycloak is running on port 8089.
  - Double-check `client_id`, `client_secret`, and `issuer-uri` values.
  - Confirm the access token is passed as `Bearer` token in the `Authorization` header.

---

## 📄 License
This project is licensed under the MIT License.
