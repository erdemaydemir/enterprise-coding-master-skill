# OAuth2 Example Project

Minimal Spring Boot OAuth2 Authorization Server example using Java 21 and Spring Authorization Server.

## Features

- OAuth2 Authorization Server endpoints
- OpenID Connect (OIDC) enabled
- In-memory registered client
- RSA key pair + JWK source for JWT signing
- In-memory user authentication
- Actuator health endpoint

## Run

```bash
mvn spring-boot:run
```

Server starts on `http://localhost:9000`.

## Test

```bash
mvn test
```
