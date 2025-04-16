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

```json
/*
Uses nodemon to run and watch changes in the `src` directory
to auto reload the app
*/
"start:dev": "nodemon src/main.ts -t ts --watch src",

/*
Uses pm2 to run production app when compiled.
It reload app when it fail.
*/
"start:prod": "pm2-runtime main.js",

/*
When the app is ready to production, it builld the app
and then copy the package.json to the build directory.
*/
"build": "nest build && copyfiles package.json prod.env dist/",
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

> Dockerfile <br>
> Production
```docker
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

# Copi the built app from the previous stage
COPY --from=build /app/dist/* .

# Install the dependencies (Just production)
RUN npm install --omit=dev
RUN npm install -g pm2

# Expose the port that the app will use
# Change <port> with the port that the app will use
EXPOSE <port>

# Run the app
CMD ["npm", "run", "start:prod"]
```

> Dockerfile.dev <br>
> Development
```docker
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

#### 3.5
In `docker-compose.yml` and `docker-compose-dev.yml` write the configuration for the docker containers

> docker-compose.yml <br>
> Production
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

> docker-compose-dev.yml <br>
> Development
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

[Back](../NestJS.md)