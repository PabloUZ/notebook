# Dockerize NestJS Application

Learn how to dockerize a NestJS application for both development and production environments.

## 1. Create Required Files

### 1.1. Create the File Structure

The Docker Compose files must live **one level above** your NestJS project, not inside it. This keeps the same structure whether you have a single application or, later on, multiple microservices side by side.

Create the following structure, with your project (created with `nest new`) nested one level below the root:

```bash
📂root-folder
├─ 📄 docker-compose.dev.yml
├─ 📄 docker-compose.yml
└─ 📂 <project name>          # nest new <project name>
   ├─ 📄 .dockerignore
   ├─ 📄 Dockerfile
   ├─ 📄 Dockerfile.dev
   ├─ 📄 .env.dev
   ├─ 📄 .env.prod
   └─ 📄 .env.example
```

> [!IMPORTANT]
> Never place `docker-compose.yml` inside the project folder. It must sit at the root, above the project, so it can build and reference it by relative path. The project keeps its **own** `.env.dev`/`.env.prod` inside its own folder — there's no shared `.env` at the root.

### 1.2. Configure `.dockerignore`

Inside `<project name>`, in `.dockerignore`, specify files that are not required for the app to work:

> \<project name\>/.dockerignore
```
node_modules/
test/
.git/
.gitignore
.eslintrc.js
.prettierrc
README.md
```

## 2. Configure Dockerfiles

### 2.1. Production Dockerfile

Inside `<project name>`, create `Dockerfile` for production environment:

> \<project name\>/Dockerfile
```Dockerfile
# Specify the image to build the container.
# This image is NodeJS based.
# More images at: https://hub.docker.com/_/node
# Change <version> with a nodejs valid version
# Use SemVer format: https://semver.org/
FROM node:<version> AS build

# Set as main working directory
WORKDIR /app

# Install copyfiles globally
RUN npm install -g copyfiles

# Copy the package.json file to the container and
# Install the dependencies
COPY package.json ./
RUN npm install

# Copy the source code to the container
COPY . .

# Build the app
RUN npm run build

# Specify the image to build the second stage.
FROM node:<version>-alpine AS production

# Set as main working directory
WORKDIR /app

# Copy the package.json from the previous stage
COPY --from=build /app/package.json .

# Install the dependencies (Just production)
RUN npm install -g pm2
RUN npm install --omit=dev

# Copy the built app from the previous stage
COPY --from=build /app/dist ./dist

# Expose the port that the app will use
# Change <port> with the port that the app will use
EXPOSE <port>

# Run the app
CMD ["npm", "run", "start:prod"]
```

**Explanation:**
- **Multi-stage build:** First stage builds the app, second stage runs it
- **Alpine image:** Smaller and more secure for production
- **PM2:** Process manager for production

### 2.2. Development Dockerfile

Inside `<project name>`, create `Dockerfile.dev` for development environment:

> \<project name\>/Dockerfile.dev
```Dockerfile
# Specify the image to build the container.
# This image is NodeJS based.
# More images at: https://hub.docker.com/_/node
# Change <version> with a nodejs valid version
# Use SemVer format: https://semver.org/
FROM node:<version>

# Set as main working directory
WORKDIR /app

# Install nodemon globally
RUN npm install -g nodemon

# Copy the package.json file to the container and
# install the dependencies
COPY package.json .
RUN npm install

# Copy the source code to the container
COPY . .

# Expose the port that the app will use
# Change <port> with the port that the app will use
EXPOSE <port>

# Run the app
CMD ["npm", "run", "start:dev"]
```

**Explanation:**
- **Nodemon:** Automatically restarts the app on code changes
- **Full Node image:** Includes development tools

## 3. Configure Docker Compose Files

These files go in `root-folder`, **next to** `<project name>`, not inside it.

### 3.1. Production Docker Compose

Create `docker-compose.yml` for production:

> docker-compose.yml
```yaml
# Production docker-compose file
# This file will run the app in production mode
services:
  # App service
  # This service will run the application
  app:
    # Image to use
    # Change <app name> with the name of the app
    image: img-<app name>:latest

    # Environment file to use
    # Points to the project's own .env file, not a root one
    env_file:
      - ./<project name>/.env.prod

    # Environment variables
    environment:
      NODE_ENV: prod

    # Container name
    # Used to identify the container
    # Also to communicate with other containers
    # Change <host name> with a name of your choice
    container_name: <host name>

    # Build the app using the previously created
    # Dockerfile
    # Change <project name> with the folder created by `nest new`
    build:
      context: ./<project name>
      dockerfile: Dockerfile

    # Port forwarding:
    # This will allow the app to be accessed from
    # the host machine.
    # Change <host port> with the port that the app will use in
    # the host machine, and <port> with the port the app listens
    # on inside the container (must match PORT in .env.prod)
    ports:
      - "<host port>:<port>"
```

### 3.2. Development Docker Compose

Create `docker-compose.dev.yml` for development:

> docker-compose.dev.yml
```yaml
# Development docker-compose file
# This file will run the app in development mode
services:
  # App service
  # This service will run the application
  app:
    # Image to use
    # Change <app name> with the name of the app
    image: img-<app name>-dev:latest

    # Environment file to use
    # Points to the project's own .env file, not a root one
    env_file:
      - ./<project name>/.env.dev

    # Environment variables
    environment:
      NODE_ENV: dev

    # Container name
    # Used to identify the container
    # Also to communicate with other containers
    # Change <host name> with a name of your choice
    container_name: <host name>

    # Build the app using the previously created
    # Dockerfile
    # Change <project name> with the folder created by `nest new`
    build:
      context: ./<project name>
      dockerfile: Dockerfile.dev

    # Volume to mount the source code
    # This will allow the app to be updated
    # without the need to rebuild the container
    volumes:
      - ./<project name>/src:/app/src

    # Port forwarding:
    # This will allow the app to be accessed from
    # the host machine.
    # Change <host port> with the port that the app will use in
    # the host machine, and <port> with the port the app listens
    # on inside the container (must match PORT in .env.dev)
    ports:
      - "<host port>:<port>"
```

**Key difference:** Development uses volumes to mount source code for hot-reloading

## 4. Configure Environment Files

### 4.1. Create Environment Variables

Inside `<project name>`, in both `.env.dev` and `.env.prod` files, add at least:

```bash
PORT=
```

> **Note:** This is the port the app listens on **inside** the container — it must match the `<port>` you set on the `ports:` mapping in the Compose file.

### 4.2. Create `.env.example`

Inside `<project name>`, create `.env.example` with the same keys as `.env.dev`/`.env.prod`, but without their values. This is the template new contributors copy to create their own env files:

> \<project name\>/.env.example
```bash
PORT=
```

### 4.3. Add to `.gitignore`

Inside `<project name>`, ignore every env file **except** the example one:

> \<project name\>/.gitignore
```
...
.env.*
!.env.example
```

## 5. Run the Application

Run these commands from `root-folder`, where `docker-compose.yml` lives.

### Development Mode
```bash
docker compose -f docker-compose.dev.yml up
```

### Production Mode
```bash
docker compose up
```
