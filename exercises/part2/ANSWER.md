## Part 2

### 2.1

Create a docker-compose.yml file that starts devopsdockeruh/simple-web-service and saves the logs into your filesystem.

```yaml
version: '3.8'

services:
  simple-web-service:
    image: devopsdockeruh/simple-web-service:ubuntu
    volumes: 
      - ./text.log:/usr/src/app/text.log
```

### 2.2

Create a docker-compose.yml and use it to start the service so that you can use it with your browser.

```yaml
version: '3.8'

services:
  simple-web-service:
    image: devopsdockeruh/simple-web-service:ubuntu
    volumes: 
      - ./text.log:/usr/src/app/text.log
    ports:
      - 8001:8080
    command: server
```

### 2.3

In the previous part we created Dockerfiles for both frontend and backend of the example application. Next, simplify the usage into one docker-compose.yml.

```yaml
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile.front
    ports:
      - 8081:5000
  backend:
    build:
      context: .
      dockerfile: Dockerfile.back
    ports:
      - 8080:8080
```

### 2.4

In this exercise you should expand the configuration done in Exercise 2.3 and set up the example backend to use the key-value database Redis.

```yaml
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile.front
    ports:
      - 8081:5000
  backend:
    build:
      context: .
      dockerfile: Dockerfile.back
    ports:
      - 8080:8080
    environment:
      REDIS_HOST: redis
  redis:
    image: redis
    restart: unless-stopped
```

### 2.5

Your task is to scale the compute containers so that the button in the application turns green.

```
docker compose up --scale compute=5
```

### 2.6

Use a Postgres database to save messages.

```
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile.front
    ports:
      - 8081:5000
  backend:
    build:
      context: .
      dockerfile: Dockerfile.back
    ports:
      - 8080:8080
    environment:
      REDIS_HOST: redis
      POSTGRES_HOST: postgres
    depends_on:
      - redis
      - postgres
  redis:
    image: redis
    restart: unless-stopped
  postgres:
    image: postgres:13.2-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    container_name: postgres
```

### 2.7

Postgres image uses a volume by default. Define manually a volume for the database in a convenient location such as in ./database so you should use now a bind mount.

```
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile.front
    ports:
      - 8081:5000
  backend:
    build:
      context: .
      dockerfile: Dockerfile.back
    ports:
      - 8080:8080
    environment:
      REDIS_HOST: redis
      POSTGRES_HOST: postgres
    depends_on:
      - redis
      - postgres
  redis:
    image: redis
    restart: unless-stopped
  postgres:
    image: postgres:13.2-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    container_name: postgres
    volumes:
      - ./data:/var/lib/postgresql/data
```

### 2.8

Add Nginx to example to work as a reverse proxy in front of the example app frontend and backend.

```
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile.front
    ports:
      - 8081:5000
    environment:
      REACT_APP_BACKEND_URL: http://localhost:80
  backend:
    build:
      context: .
      dockerfile: Dockerfile.back
    ports:
      - 8080:8080
    environment:
      REDIS_HOST: redis
      POSTGRES_HOST: postgres
      REQUEST_ORIGIN: http://localhost:80
    depends_on:
      - redis
      - postgres
  redis:
    image: redis
    restart: unless-stopped
  postgres:
    image: postgres:13.2-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    container_name: postgres
  nginx:
    container_name: nginx
    image: nginx
    ports:
      - 80:80
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
```

### 2.9

Most of the buttons may have stopped working in the example application. Make sure that every button for exercises works.

Add proxy for /ping/ and route it to /ping/ endpoint

```
events { worker_connections 1024; }

http {
  server {
    listen 80;

    location / {
      proxy_pass http://frontend:5000/;
    }

    # configure here where requests to http://localhost/api/...
    # are forwarded
    location /api/ {
      proxy_set_header Host $host;
      proxy_pass http://backend:8080/;
    }

    location /ping/ {
      proxy_set_header Host $host;
      proxy_pass http://backend:8080/ping;
    }
  }
}
```