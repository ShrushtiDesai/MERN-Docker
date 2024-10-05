# Full Stack Application with Docker

This repository contains the Docker setup for both frontend and backend applications. The project leverages `Dockerfile` for each part and a `docker-compose.yml` file to orchestrate the services.

## Prerequisites
### Ubuntu - Install through Terminal
Ensure you have the following installed:
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
### Windows - Download Docker Desktop
---

## Steps to run the project

### 1. Create a `Dockerfile` for the Frontend
This Dockerfile sets up a lightweight Node.js environment, installs dependencies, copies the app code, exposes port 5173, and runs the app in development mode.
In the `frontend` folder, create a `Dockerfile` with the following contents:

```Dockerfile
FROM node:20-alpine3.18

# RUN addgroup app && adduser -S -G app app

# USER app

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5173

CMD npm run dev
```
### 2. Create a `Dockerfile` for the Backend
- This Dockerfile enhances security by running the application as a non-root app user.
- Ownership of the /app directory is explicitly changed to app to allow the user to have proper permissions.
- The app is exposed on port 8000 and started using the npm start command.
In the `Backend` folder, create a `Dockerfile` with the following contents:

```Dockerfile
FROM node:20-alpine3.18

RUN addgroup app && adduser -S -G app app

USER app

WORKDIR /app

COPY package*.json ./

# change ownership of the /app directory to the app user
USER root

# change ownership of the /app directory to the app user
# chown -R <user>:<group> <directory>
# chown command changes the user and/or group ownership of for given file.
RUN chown -R app:app .

# change the user back to the app user
USER app

RUN npm install

COPY . . 
EXPOSE 8000 
CMD npm start
```
### 3. Create a `Compose.yaml` in Root Directory.
This docker-compose.yml sets up a full-stack environment consisting of:

- A frontend service (web) using Vite, which watches for changes and syncs files during development.
- A backend service (api) that interacts with a MongoDB database, also watching for changes during development.
- A MongoDB service (db) to handle the database, with persistent storage in a Docker volume.
Compose.yaml contents:
```
version: "3.8"

services:
  web:
    depends_on: 
      - api
    build: ./frontend
    ports:
      - 5173:5173

    environment:
      VITE_API_URL: http://localhost:8000

    develop:
      watch:
        # watch for changes
        - path: ./frontend/package.json
          action: rebuild
        - path: ./frontend/package-lock.json
          action: rebuild
        # it'll watch for changes and sync in real time
        - path: ./frontend
          target: /app
          action: sync

  # define the api service/container
  api: 
    depends_on: 
      - db
    build: ./backend
    ports: 
      - 8000:8000
    environment: 
      DB_URL: mongodb://db/anime
   
    develop:
      watch: 
        - path: ./backend/package.json
          action: rebuild
        - path: ./backend/package-lock.json
          action: rebuild
        
        # sync the changes
        - path: ./backend
          target: /app
          action: sync

  # define the db service
  db:
    image: mongo:latest
    ports:
      - 27017:27017
    volumes:
      - anime:/data/db

volumes:
  anime:
```
### Run the following command
```bash
sudo docker compose up
```
### To allow the changes to sync in real time run the following command
```bash
sudo docker compose watch
```
The MERN app is now running at localhost 5173 and sync to changes in real time!
