# Multi-Tier App

A containirized multi tier application orchestrated using Docker Compose and deplyed on an AWS EC2 (ubuntu) instance. This project demonstrate service discovery, bridge networking, and persistence data storage in a microservice setup.

## Architecture

Browser → Frontend (Nginx) → Backend API (Node.js) → MongoDB database
```
| Service  | Description                      | Port  | Internal Host Mapping
|----------|----------------------------------|-------|
| frontend | Web UI server via Nginx          | 80    | Frontend
| backend  | REST API service                 | 80    | Backend
| database | MongoDB (custom container image) | 27017 | Database
```
## Tech Stack
- Docker & Docker Compose
- MongoDB
- Backend / Frontend
- AWS EC2 (Ubuntu Linux)

## Project Structure
```
multi-tier-app/
├── docker-compose.yml
├── backend/
│   └── Dockerfile
├── database/
│   └── Dockerfile
└── frontend/
    └── Dockerfile
```
## Configuration File Reference
1. docker-compose.yml
```
services:
  database:
    build: ./database
    container_name: db
    restart: always
    networks:
      - backend-network
    volumes:
      - mongo-data:/data/db
    ports:
      - "27017:27017"

  backend:
    build: ./backend
    container_name: api
    restart: always
    depends_on:
      - database
    networks:
      - backend-network
      - frontend-network
    ports:
      - "80"

  frontend:
    build: ./frontend
    container_name: ui
    restart: always
    depends_on:
      - backend
    networks:
      - frontend-network
    ports:
      - "80"

networks:
  backend-network:
  frontend-network:

volumes:
  mongo-data:
```
2. Service Dockerfiles
- database/Dockerfile
```
# Use the official MongoDB image
FROM mongo:latest
# Expose MongoDB's default port
EXPOSE 27017
```
- backend/Dockerfile
```
# Use the official Node.js image
FROM node:18
# Set working directory
WORKDIR /app
# Copy package.json and install dependencies
COPY package*.json ./
RUN npm init -y && npm install express mongoose cors body-parser
# Copy application code
COPY . .
# Create index.js with Express API and MongoDB connection
RUN echo "const express = require('express'); \
const mongoose = require('mongoose'); \
const app = express(); \
const PORT = 80; \
mongoose.connect('mongodb://db:27017/mydb', { useNewUrlParser: true }) \
  .then(() => console.log('Connected to MongoDB')) \
  .catch(err => console.log('MongoDB connection error:', err)); \
app.get('/', (req, res) => res.send('Hello from Backend')); \
app.listen(PORT, () => console.log('Backend running on port ' + PORT));" > index.js
# Expose port 80
EXPOSE 80
# Run the application
CMD ["node", "index.js"]
```
- frontend/Dockerfile
```
# Use the official lightweight Nginx image
FROM nginx:alpine
# Expose port 80 for incoming traffic
EXPOSE 80
# Start Nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]
```
## Getting Started
### Prerequisites & Infrastructure Setup
- Docker & Docker Compose installed on your EC2 instance.   
- AWS EC2 Security Group Rules: Inbound rules HTTP/TCP traffic on Port 80.
- Storage Allocation: An EBS root volume of at least 15–20 GB to prevent build failures from disk exhaustion.

1. Navigate to Project Directory:
```
cd multi-tier-app
```
2. Build and Run the Containers in Detached Mode:
```
docker compose up build -d
```
3. Verify Deployment:
```
docker compose ps
```
4. Access the Application:
Open your browser and navigate to http://<ec2-public-ip> to view the application front end.
## Useful Operational Commands
```
# Stream logs for a specific service
docker compose logs -f (backend/frontend)

# Stop and remove containers
docker compose down

# Stop containers and wipe persistent database volumes
docker compose down -v
```
## Troubleshooting Guide

- no space left on device during build:
 Enlarge your AWS EBS volume in the AWS Management Console, then expand the partition and filesystem on your instance:
```
sudo growpart /dev/nvme0n1 1
sudo resize2fs /dev/nvme0n1p1
```
- Backend can't connect to the database:
  Verify your connection string uses the container name (mongodb://db:27017/mydb) rather than localhost, since containers communicate across isolated Docker bridge networks.
- Site not loading in browser:
  Confirm that port 80 is open in your AWS EC2 Security Group inbound rules and `docker ps`.

