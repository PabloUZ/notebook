[← Back to index](../index.md)

---

# Dockerize NestJS Application

In this step, you will learn how to dockerize your NestJS application for both development and production environments.

---

## 1. Create Required Files

### 1.1. Create the File Structure

Create the following structure in your project root:

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

### 1.2. Configure `.dockerignore`

In `.dockerignore`, specify files that are not required for the app to work:

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

---

## 2. Configure Dockerfiles

### 2.1. Production Dockerfile

Create `Dockerfile` for production environment:

> Dockerfile
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

# Copy the package.json and the environment file from the previous stage
COPY --from=build /app/prod.env .
COPY --from=build /app/package.json .

# Install the dependencies (Just production)
RUN npm install -g pm2
RUN npm install --omit=dev
RUN npm install dotenv-cli

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

Create `Dockerfile.dev` for development environment:

> Dockerfile.dev
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

---

## 3. Configure Docker Compose Files

### 3.1. Production Docker Compose

Create `docker-compose.yml` for production:

> docker-compose.yml
```yaml
# Production docker-compose file
# This file will run the app in production mode
# ${} are environment variables that will be replaced
# with the values in the .env file
services:
  # App service
  # This service will run the application
  app:
    # Image to use
    # Change <app name> with the name of the app
    image: img-<app name>:latest

    # Environment file to use
    env_file:
      - ./prod.env

    # Environment variables
    environment:
      NODE_ENV: ${NODE_ENV}

    # Container name
    # Used to identify the container
    # Also to communicate with other containers
    container_name: ${HOST_NAME}

    # Build the app using the previously created
    # Dockerfile
    build:
      context: .
      dockerfile: Dockerfile

    # Port forwarding:
    # This will allow the app to be accessed from
    # the host machine.
    # Change <host port> with the port that the app
    # will use in the host machine
    ports:
      - "<host port>:${PORT}"
```

### 3.2. Development Docker Compose

Create `docker-compose-dev.yml` for development:

> docker-compose-dev.yml
```yaml
# Development docker-compose file
# This file will run the app in development mode
# ${} are environment variables that will be replaced
# with the values in the .env file
services:
  # App service
  # This service will run the application
  app:
    # Image to use
    # Change <app name> with the name of the app
    image: img-<app name>-dev:latest

    # Environment file to use
    env_file:
      - ./dev.env

    # Environment variables
    environment:
      NODE_ENV: ${NODE_ENV}

    # Container name
    # Used to identify the container
    # Also to communicate with other containers
    container_name: ${HOST_NAME}

    # Build the app using the previously created
    # Dockerfile
    build:
      context: .
      dockerfile: Dockerfile.dev

    # Volume to mount the source code
    # This will allow the app to be updated
    # without the need to rebuild the container
    volumes:
      - ./src:/app/src

    # Port forwarding:
    # This will allow the app to be accessed from
    # the host machine.
    # Change <host port> with the port that the app
    # will use in the host machine
    ports:
      - "<host port>:${PORT}"
```

**Key difference:** Development uses volumes to mount source code for hot-reloading

---

## 4. Configure Environment Files

### 4.1. Create Environment Variables

In both `dev.env` and `prod.env` files, add at least:

```bash
NODE_ENV=
HOST_NAME=
PORT=
```

### 4.2. Add to `.gitignore`

Add the environment files to `.gitignore`:

```
...
prod.env
dev.env
```

---

## 5. Run the Application

### Development Mode
```bash
docker-compose -f docker-compose-dev.yml up
```

### Production Mode
```bash
docker-compose up
```

---

## Next Steps

Now that your application is dockerized, you need to configure environment variables properly using NestJS ConfigModule.

**Continue with:** [Environment Variables](./environment-variables.md)

---

[← Back to index](../index.md)
