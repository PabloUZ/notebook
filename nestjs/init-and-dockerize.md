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
├─ 📄 .dockerignore
├─ 📄 docker-compose-dev.yml
├─ 📄 docker-compose.yml
├─ 📄 Dockerfile
├─ 📄 Dockerfile.dev
├─ 📄 dev.env
└─ 📄 prod.env
```
#### 3.2
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

#### 3.3
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

CMD ["npm", "run", "start:prod"]
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

CMD ["npm", "run", "start:prod"]
```

#### 3.5
In `docker-compose.yml` and `docker-compose-dev.yml` write the configuration for the docker containers

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
      - "<host port>:${PORT}"
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
      - "<host port>:${PORT}"
```

#### 3.6
In the `.env` files write the following variables at least

```bash
NODE_ENV=
HOST_NAME=
PORT=
```

### 4. Add the .env files to .gitignore
```
...
prod.env
dev.env
```