[Back](../NestJS.md)

# NestJS environment settings

Instructions to set up the NestJS environment using Docker for development and production.


### 1. Create a new project
To create a new project with NestJS just run the following command:

```bash
nest new <project name>
```

### 2. Configure `package.json` scripts
Create 3 scripts in `package.json`:

- `start:dev`
- `start:prod`
- `build`

Then add the following commands:

```bash
"start:dev": "nodemon src/main.ts -t ts --watch src"
"start:prod": "pm2-runtime main.js"
"build": "nest build && copyfiles package.json prod.env dist/"
```

### 3. Create the new files
#### 3.1
Create the following structure:

```bash
📂my-project
├─ 📂 init
│   ├─ 📄 init-dev.sh
│   └─ 📄 init-prod.sh
├─ 📄 .dockerignore
├─ 📄 docker-compose-dev.yml
├─ 📄 docker-compose.yml
├─ 📄 Dockerfile
├─ 📄 Dockerfile.dev
├─ 📄 dev.env
└─ 📄 prod.env
```

#### 3.2
In the init files execute the script previously configured in `package.json`:

> init-dev.sh
```bash
#!/bin/bash

sleep 20

npm run start:dev
```

> init-prod.sh
```bash
#!/bin/bash

sleep 20

npm run start:prod
```

The command `sleep` is necesary to set timeout until database start

#### 3.3
In `.dockerignore` write the files that are not required for the app to work.

> .dockerignore
```
node_modules/
test/
.git/
.gitignore
.eslintrc.js
.prettierrc
README.md
```

#### 3.4
In `Dockerfile` and `Dockerfile.dev` write the configuration for the docker images

> Dockerfile
```docker
FROM node:<version>

RUN mkdir /app_temp

WORKDIR /app_temp

RUN npm install -g pm2 copyfiles

COPY package.json .
RUN npm install

COPY . .

RUN npm run build

RUN mkdir -p /app_temp/dist/init

RUN cp ./init/init-prod.sh /app_temp/dist/init

RUN rm -rf /app

RUN mv /app_temp/dist/ /app

WORKDIR /app

RUN npm install

RUN rm -rf /app_temp

EXPOSE <port>

CMD ["bash", "init/init-prod.sh"]
```

> Dockerfile.dev
```docker
FROM node:<version>

WORKDIR /app

RUN npm install -g nodemon

COPY package.json .
RUN npm install

COPY . .

EXPOSE <port>

CMD ["bash", "init/init-dev.sh"]
```

#### 3.5
In `docker-compose.yml` and `dockercompose-dev.yml` write the configuration for the docker containers

> docker-compose.yml
```yaml
services:
  app:
    image: img-<app name>:latest
    env_file:
      - ./prod.env
    environment:
      NODE_ENV: ${NODE_ENV}
    container_name: ${HOST_NAME}
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "<host port>:<container port>"
    depends_on:
      - db
  db:
    image: mysql
    env_file:
      - ./prod.env
    container_name: ${DB_HOST_NAME}
    environment:
      MYSQL_ROOT_PASSWORD: ${DATABASE_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DATABASE_NAME}
      MYSQL_USER: ${DATABASE_USER}
      MYSQL_PASSWORD: ${DATABASE_PASSWORD}
```

> docker-compose-dev.yml
```yaml
services:
  app:
    image: img-<app name>-dev:latest
    env_file:
      - ./dev.env
    environment:
      NODE_ENV: ${NODE_ENV}
    container_name: ${HOST_NAME}
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - ./src:/app/src
    ports:
      - "<host port>:<container port>"
    depends_on:
      - db
  db:
    image: mysql
    env_file:
      - ./dev.env
    container_name: ${DB_HOST_NAME}
    environment:
      MYSQL_ROOT_PASSWORD: ${DATABASE_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DATABASE_NAME}
      MYSQL_USER: ${DATABASE_USER}
      MYSQL_PASSWORD: ${DATABASE_PASSWORD}
```

#### 3.6
In the `.env` files write the following variables at least

```bash
NODE_ENV=
HOST_NAME=
DB_HOST_NAME=
DATABASE_ROOT_PASSWORD=
DATABASE_NAME=
DATABASE_USER=
DATABASE_PASSWORD=
```

### 4. Add the .env files to .gitignore
```
...
prod.env
dev.env
```