# OAuth2 Example Authorization Server

This folder contains a minimal Spring Boot OAuth2 Authorization Server sample with OpenID Connect enabled.

## Stack
- Java 17+
- Spring Boot 3.3.x
- Spring Authorization Server

## Quick run
```bash
cd oauth2-example
mvn spring-boot:run
```

Server runs on `http://localhost:9000`.

## Demo credentials
- User: `user`
- Password: `password`
- Client ID: `demo-client`
- Client Secret: `demo-secret`

## Useful endpoints
- OpenID configuration: `http://localhost:9000/.well-known/openid-configuration`
- JWK set: `http://localhost:9000/oauth2/jwks`
- Health: `http://localhost:9000/actuator/health`
