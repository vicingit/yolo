# Project Name

A step-by-step procedure to containerize this application using Docker, setting up a microservice architecture.

## Table of Contents

1. [Overview](#overview)
2. [Requirements](#requirements)
3. [Containerize the Client/Frontend](#containerize-the-clientfrontend)
4. [Containerize the Backend](#containerize-the-backend)
5. [Configure Microservice Architecture using Docker Compose](#configure-microservice-architecture-using-docker-compose)
6. [Ensure Connectivity Between Containers](#ensure-connectivity-between-containers)
7. [Build and Start Your Microservices](#build-and-start-your-microservices)
8. [Stop the Application](#stop-the-application)
9. [Contact](#contact)

## Overview

This file outlines a step-by-step procedure of how this application was containerized using Docker.

## Requirements

Make sure that you have the following installed:
- [Docker](https://docs.docker.com/engine/install/)

## Containerize the Client/Frontend

1. **Navigate to the client directory:**

    ```bash
    cd client
    ```

2. **Create a Dockerfile:**

    ```bash
    touch Dockerfile
    ```

3. **Configure the Dockerfile using a multi-stage build to ensure that the resultant image is lightweight:**

    ```dockerfile
    # Build stage
    FROM node:14-alpine3.16 as build

    # Set the working directory inside the container
    WORKDIR /app

    # Copy package.json and package-lock.json to install dependencies
    COPY package*.json .

    # Install npm dependencies
    RUN npm install

    # Copy the rest of the client application code to the working directory
    COPY . .

    # Build the application and remove development dependencies
    RUN npm run build && \
        npm prune --production

    # Production stage
    FROM node:16-alpine3.16 as production-stage

    # Set the working directory inside the container
    WORKDIR /app

    # Copy the built app from the build stage to the production container
    COPY --from=build /app /app

    # Expose port 3000 to communicate with the outside world
    EXPOSE 3000

    # Command to start the application
    CMD ["npm", "start"]
    ```

4. **Create a `.dockerignore` file:**

    ```bash
    touch .dockerignore
    ```

5. **In the `.dockerignore` file, specify which files to exclude when building the Docker image:**

    ```plaintext
    node_modules
    ```

## Containerize the Backend

1. **Navigate to the backend directory:**

    ```bash
    cd backend
    ```

2. **Create a Dockerfile:**

    ```bash
    touch Dockerfile
    ```

3. **Configure the Dockerfile using a multi-stage build to ensure that the resultant image is lightweight:**

    ```dockerfile
    FROM node:14-alpine3.16

    # Set the working directory to /app inside the container
    WORKDIR /app

    # Copy package.json and package-lock.json to the working directory
    COPY package*.json ./

    # Install npm dependencies
    RUN npm install

    # Copy the entire application code to the working directory
    COPY . .

    # Expose the port on which the Node.js application will run
    EXPOSE 5000

    # Command to start the Node.js application
    CMD ["npm", "start"]
    ```

4. **Create a `.dockerignore` file:**

    ```bash
    touch .dockerignore
    ```

5. **In the `.dockerignore` file, specify which files to exclude when building the Docker image:**

    ```plaintext
    node_modules
    ```

## Configure Microservice Architecture using Docker Compose

1. **Navigate to the root directory of the repository and create a `docker-compose.yaml` file:**

    ```bash
    touch docker-compose.yaml
    ```

2. **Configure the `docker-compose.yaml` file as follows:**

    ```yaml
    version: '3.8'

    services:
      # Backend service configuration
      backend:
        build: ./backend  # Build the backend service using the Dockerfile in the ./backend directory
        image: estheruge/backend-service:v1.0.0  # Tag the built image with a specific version
        container_name: backend_container  # Name of the backend container
        environment:
          - MONGODB_URI=mongodb://mongo:27017/yolomy  # Set the MongoDB connection URI for the backend
        ports:
          - "5000:5000"  # Map port 5000 of the container to port 5000 on the host
        networks:
          - custom_network  # Attach backend service to the custom network
        depends_on:
          - mongo  # Ensure MongoDB service is started before the backend service

      # Client service configuration
      client:
        build: ./client  # Build the client service using the Dockerfile in the ./client directory
        image: estheruge/client-service:v1.0.0  # Tag the built image with a specific version
        container_name: client_container  # Name of the client container
        environment:
          - NODE_ENV=production  # Set Node.js environment to production
        ports:
          - "3000:3000"  # Map port 3000 of the container to port 3000 on the host
        depends_on:
          - backend  # Ensure backend service is started before the client service
        networks:
          - custom_network  # Attach client service to the custom network
        stdin_open: true  # Keep stdin open so container can accept input
        tty: true  # Allocate a pseudo-TTY for the container, useful for debugging

      # MongoDB service configuration
      mongo:
        image: mongo:latest  # Use the latest MongoDB Docker image
        container_name: mongo_container  # Name of the MongoDB container
        networks:
          - custom_network  # Attach MongoDB service to the custom network
        ports:
          - "27017:27017"  # Map port 27017 of the container to port 27017 on the host
        volumes:
          - mongo_data:/data/db  # Mount a volume named 'mongo_data' for MongoDB data persistence

    networks:
      custom_network:
        driver: bridge  # Use the bridge network driver for the custom network

    volumes:
      mongo_data:
        driver: local  # Use the local volume driver for 'mongo_data' volume
    ```

## Ensure Connectivity Between Containers

1. **Navigate to the `server.js` file in the backend folder and change the `mongodb_url` to point to your MongoDB container instead of `localhost`.**

    ```javascript
    // Connecting to the Database
    let mongodb_url = 'mongodb://127.0.0.1';
    dbName = 'yolomy';
    ```

    Update to:

    ```yaml
    environment:
      - MONGODB_URI=mongodb://mongo:27017/yolomy  # Set the MongoDB connection URI for the backend
    ```

## Build and Start Your Microservices

1. **Navigate to the root folder of the repository (where your `docker-compose.yaml` file is located) and initialize your microservice application using the following command:**

    ```bash
    sudo docker compose up
    ```

## Stop the Application

1. **In the same terminal where you launched your microservices, press `Ctrl+C` to stop the application and terminate it using the following command:**

    ```bash
    sudo docker compose down
    ```

