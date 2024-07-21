# YOLO APP

## 1. Base Image Selection

Alpine Linux was chosen as the base image for its lightweight nature and strong security features. Known for minimalism, it ensures smaller Docker image sizes and faster builds. Alpine's wide support and compatibility also make it ideal for efficient containerized applications, prioritizing both performance and security.
# Explanation of Implementation Choices

This document explains the reasoning behind various implementation choices made during the process of containerizing the application and setting up a microservice architecture.

## 1. Choice of the Base Image

### Client/Frontend

- **Base Image:** `node:14-alpine3.16`
- **Reasoning:** The Alpine version of Node.js is chosen for its lightweight nature, which ensures smaller image sizes and faster download and startup times. Version 14 is used for compatibility with the existing application code.

### Backend

- **Base Image:** `node:14-alpine3.16`
- **Reasoning:** Similar to the frontend, the lightweight Alpine version of Node.js is chosen. Version 14 is used for consistency across the project and compatibility with the backend application code.

### MongoDB

- **Base Image:** `mongo:latest`
- **Reasoning:** The latest version of MongoDB is chosen to ensure the use of the most up-to-date features and security patches.

## 2. Dockerfile Directives

### Client/Frontend

- **Multi-Stage Build:**
  - **First Stage (Build):** Uses `node:14-alpine3.16` to install dependencies and build the application.
  - **Second Stage (Production):** Uses `node:16-alpine3.16` to run the application with only production dependencies.
- **Directives:**
  - `WORKDIR /app`: Sets the working directory inside the container.
  - `COPY package*.json .`: Copies package files to install dependencies.
  - `RUN npm install`: Installs dependencies.
  - `COPY . .`: Copies the rest of the application code.
  - `RUN npm run build && npm prune --production`: Builds the application and removes development dependencies.
  - `EXPOSE 3000`: Exposes port 3000 for communication.
  - `CMD ["npm", "start"]`: Starts the application.

### Backend

- **Single-Stage Build:**
  - Uses `node:14-alpine3.16` to install dependencies and run the application.
- **Directives:**
  - `WORKDIR /app`: Sets the working directory inside the container.
  - `COPY package*.json ./`: Copies package files to install dependencies.
  - `RUN npm install`: Installs dependencies.
  - `COPY . .`: Copies the rest of the application code.
  - `EXPOSE 5000`: Exposes port 5000 for communication.
  - `CMD ["npm", "start"]`: Starts the application.

## 3. Docker-Compose Networking

- **Application Port Allocation:**
  - Backend: Maps container port 5000 to host port 5000.
  - Client: Maps container port 3000 to host port 3000.
  - MongoDB: Maps container port 27017 to host port 27017.
- **Bridge Network Implementation:**
  - A custom bridge network named `custom_network` is created to enable communication between the services.

## 4. Docker-Compose Volume Definition and Usage

- **MongoDB Volume:**
  - A volume named `mongo_data` is created to persist MongoDB data.
  - Ensures data persistence across container restarts.

## 5. Git Workflow

- **Branching Strategy:**
  - `main` branch for production-ready code.
  - `dev` branch for ongoing development.
  - Feature branches (`feature/branch-name`) for individual features and fixes.
- **Commit Messages:**
  - Use descriptive commit messages for clarity and traceability.
- **Pull Requests:**
  - Feature branches are merged into `dev` via pull requests, which are reviewed and tested before merging.
- **CI/CD:**
  - Implemented using GitHub Actions to automate the build, test, and deployment processes.

## 6. Successful Running and Debugging Measures

- **Running the Applications:**
  - Use `sudo docker compose up` to start all services.
  - Verify services are running by accessing the client and backend endpoints.
- **Debugging Measures:**
  - Check container logs using `docker logs <container_name>`.
  - Use `docker exec -it <container_name> /bin/sh` to access the container shell for manual debugging.
  - Ensure correct environment variable configurations.

## 7. Good Practices

- **Docker Image Tag Naming Standards:**
  - Use semantic versioning for image tags (e.g., `v1.0.0`).
  - Include the service name in the image tag for clarity (e.g., `victore/backend-service:v1.0.0`).
- **Docker Compose:**
  - Use environment variables for sensitive data and configurations.
  - Ensure proper port mappings and network configurations for seamless service communication.


## 2. Dockerfile Directives

### Dockerfile Creation and Running Directives

The Dockerfile includes directives to define the build process for each service. Key directives used are:

- **FROM:** Specifies the base image for the service.
- **WORKDIR:** Sets the working directory inside the container.
- **COPY:** Copies files from the host machine to the container.
- **RUN:** Executes commands during the build process.
- **CMD:** Specifies the command to run when the container starts.
- **EXPOSE:** Exposes ports from the container to the host machine.

## 3. Docker-Compose Networking

### Application Port Allocation and Bridge Network Implementation

Docker Compose is used to manage multi-container applications. Networking and port allocation are configured in the `docker-compose.yml` file:

```
version: '3.8'

services:
  # Backend service configuration
  backend:
    build: ./backend
    image: victore/backend-service:v1.0.0
    container_name: backend_container
    environment:
      - MONGODB_URI=mongodb://mongo:27017/yolomy
    ports:
      - "5000:5000"
    networks:
      - custom_network
    depends_on:
      - mongo

  # Client service configuration
  client:
    build: ./client
    image: victore/client-service:v1.0.0
    container_name: client_container
    environment:
      - NODE_ENV=production
    ports:
      - "3000:3000"
    depends_on:
      - backend
    networks:
      - custom_network
    stdin_open: true
    tty: true

  # MongoDB service configuration
  mongo:
    image: mongo:latest
    container_name: mongo_container
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    networks:
      - custom_network

networks:
  custom_network:
    driver: bridge

volumes:
  mongo_data:
    driver: local

```

#### In this setup:

- Ports 5000, 3000, and 27017 are mapped between the host and containers.
- A custom bridge network `custom_network` connects all services for internal communication.

## 4. Docker-Compose Volume Definition and Usage

- Volume Definition and Usage

Volumes ensure persistent data storage. In the `docker-compose.yml`:

```
volumes:
  mongo_data:
    driver: local
```

## 5 .Git Workflow

Git Workflow Used
- Cloning the Repository: Clone the project repository to local machines using the `git clone` command: `https://github.com/Essyuge/yolo.git`.
- Making Changes: Make changes to files locally, including implementing new features, fixing bugs, or making enhancements.
- Committing Changes: After making changes, commit them directly to the master branch using the `git add <filename>` and `git commit -m"commit message"` commands.
- Pushing Changes: Once changes are committed locally, push them to the remote repository using the `git push` command.

## 6. Successful Running of Applications and Debugging Measures

- Deployment and Debugging
- Successful Deployment: Ensure Docker and Docker Compose are installed.
- Debugging Measures: Use:
  - `docker build -t <imagename:version>` to build an image.
  - `docker images` to check images built.
  - `docker ps` to check container status.
  - `docker login` to log in to DockerHub.
  - `docker push <username:image>` to deploy image to DockerHub.
  - `docker-compose down` followed by `docker-compose up` for restarts.
  - `docker-compose logs <service>` for logs.

## 7. Docker Image Tag Naming Standards

- Docker Image Tag Naming Standards
- Images are tagged using semantic versioning (`v1.0.0`):
  - `victore/backend-service:v1.0.0`
  - `victore/client-service:v1.0.0`




