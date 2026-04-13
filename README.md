## DEV

1. Clonar el repositorio
2. Crear un .env basado en el .env.template
3. Ejecutar el comando `git submodule update --init --recursive` para reconstruir los submodulos
4. Ejecutar el comando `docker compose up --build`


### Pasos para crear los Git Submodules

1. Crear un nuevo repositorio en GitHub
2. Clonar el repositorio en la máquina local
3. Añadir el submodule, donde `repository_url` es la url del repositorio y `directory_name` es el nombre de la carpeta donde quieres que se guarde el sub-módulo (no debe de existir en el proyecto)
```
git submodule add <repository_url> <directory_name>
```
4. Añadir los cambios al repositorio (git add, git commit, git push)
Ej:
```
git add .
git commit -m "Add submodule"
git push
```
5. Inicializar y actualizar Sub-módulos, cuando alguien clona el repositorio por primera vez, debe de ejecutar el siguiente comando para inicializar y actualizar los sub-módulos
```
git submodule update --init --recursive
```
6. Para actualizar las referencias de los sub-módulos
```
git submodule update --remote
```


## Importante
Si se trabaja en el repositorio que tiene los sub-módulos, **primero actualizar y hacer push** en el sub-módulo y **después** en el repositorio principal. 

Si se hace al revés, se perderán las referencias de los sub-módulos en el repositorio principal y tendremos que resolver conflictos.


## Compose para DEV y pruebas

```bash


version: '3'


services:

  nats-server:
    image: nats:latest
    ports:
      - 8222:8222

  client-gateway:
    build: ./client-gateway
    ports:
      - ${CLIENT_GATEWAY_PORT}:3000
    volumes:
      - ./client-gateway/src:/usr/src/app/src
    command: npm run start:dev
    environment:
      - PORT=3000
      - NATS_SERVERS=nats://nats-server:4222
  
  products-ms:
    build: ./products-ms
    ports:
      - 3001:3001
    volumes:
      - ./products-ms/src:/usr/src/app/src
    command: npm run start:dev
    environment:
      - PORT=3001
      - NATS_SERVERS=nats://nats-server:4222
      - DATABASE_URL=file:./dev.db

  orders-ms:
    build: ./orders-ms
    ports:
      - 3002:3002
    volumes:
      - ./orders-ms/src:/usr/src/app/src
    command: npm run start:dev
    environment:
      - PORT=3002
      - NATS_SERVERS=nats://nats-server:4222
      - DATABASE_URL=postgresql://postgres:123456@orders-db:5432/ordersdb?schema=public
    depends_on:
      - orders-db
  
  orders-db:
    container_name: orders_database
    image: postgres:16.2
    restart: always
    volumes:
      - ./orders-ms/postgres:/var/lib/postgresql/data
    ports:
      - 5435:5432
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=123456
      - POSTGRES_DB=ordersdb

  ## ===============
  # Payments Microservice
  ## ===============

  payments-ms:
    build: ./payments-ms
    ports:
      - ${PAYMENTS_MS_PORT}:${PAYMENTS_MS_PORT}
    volumes:
      - ./payments-ms/src:/usr/src/app/src
    command: npm run start:dev
    environment:
      - PORT=${PAYMENTS_MS_PORT}
      - NATS_SERVERS=nats://nats-server:4222
      - STRIPE_SECRET=${STRIPE_SECRET}
      - STRIPE_ENDPOINT_SECRET=${STRIPE_ENDPOINT_SECRET}
      - STRIPE_SUCCESS_URL=${STRIPE_SUCCESS_URL}
      - STRIPE_CANCEL_URL=${STRIPE_CANCEL_URL}

  auth-ms:
    build: ./auth-ms
    ports:
      - ${AUTH_MS_PORT}:${AUTH_MS_PORT}
    volumes:
      - ./auth-ms/src:/usr/src/app/src
    command: npm run start:dev
    environment:
      - PORT=${AUTH_MS_PORT}
      - NATS_SERVERS=nats://nats-server:4222
      - DATABASE_URL=${AUTH_DATABASE_URL}
      - JWT_SECRET=${JWT_SECRET}


```


## PROD

1. Clonar repositorio
2. ``````