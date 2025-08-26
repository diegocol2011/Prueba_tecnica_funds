# BTG Funds Platform (Spring Boot + MongoDB)

Prueba técnica – Back End (Spring Boot).

## Stack
- Java 17, Spring Boot 3.3
- MongoDB
- JWT (stateless)
- Pruebas unitarias (JUnit + Mockito)
- Swagger/OpenAPI
- Docker & docker-compose
- CloudFormation (ECS Fargate – skeleton)

## Ejecutar localmente
1. **Levantar MongoDB y la app con Docker Compose:**
   ```bash
   mvn -q -DskipTests package
   docker compose up --build
   ```

2. **Variables:** `MONGODB_URI`, `JWT_SECRET` (ver `application.yml`).

3. **Swagger:** http://localhost:8080/swagger-ui.html

## Flujo funcional
- Registro: `POST http://localhost:8080/auth/register`  → devuelve JWT. Balance inicial = 500,000 COP.
  con body `{
  "name": "Diego",
  "email": "diego@example.com",
  "password": "MiClaveSegura123!",
  "preferredChannel": "EMAIL",
  "phone": "+573001234567"
  }`

- Login: `POST http://localhost:8080/auth/login` → devuelve JWT.

- Listar fondos: `GET http://localhost:8080/funds` (público para demo, puedes cerrarlo en `SecurityConfig`).

- Suscribirse: `POST http://localhost:8080/funds/subscriptions` 
  con body `{ "fundId": "1", "amount": 100000 }` y header `Authorization: Bearer <token>`.

- Cancelar: `DELETE http://localhost:8080/funds/subscriptions/{id}`

- Historial: `GET http://localhost:8080/funds/transactions`

> Regla de negocio: Si no hay saldo suficiente, se retorna:  
> `No tiene saldo disponible para vincularse al fondo <Nombre del fondo>`.

## Seguridad
- JWT HS256. Cambia `app.jwt.secret` en `application.yml` o variable `JWT_SECRET`.
- Roles: por defecto `USER`. Puedes añadir `ADMIN` directamente en la colección `clients`.

## CloudFormation
- Plantilla en `infra/cloudformation.yml` (ECS Fargate). Requiere:
  - Imagen subida a ECR (`EcrImageUri`).
  - VPC y Subnets existentes.
- Paso a paso:
  1. `mvn -DskipTests package`
  2. Construye y sube imagen a ECR.
  3. Lanza el stack de CloudFormation con parámetros.

## Tests
```bash
mvn test
```

## Notas
- `DataLoader` carga los fondos del enunciado si la colección está vacía.
- Para simplificar, el controlador usa `username` del principal como **id**; en proyecto real,
  se propaga el `clientId` real desde el token con un `AuthenticationPrincipal` custom o un `HandlerMethodArgumentResolver`.