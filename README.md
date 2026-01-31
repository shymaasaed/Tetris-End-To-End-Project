### 🎮 Tetris Project – Local Setup Guide (Node 14) : 
This project consists of:
- React Frontend (served by Nginx)
- Node.js Backend API (Node 14)
- Redis (Leaderboard storage)
All services run in Docker containers connected via one Docker network.
## ✅ Prerequisites
Make sure you have installed:
```git
docker
node (14.x)
npm

```
Check versions:
```git --version
docker --version
node -v
npm -v

```
⚠️ This project is tested with Node.js 14.x.
Using another version may cause build issues.
## 📁 Project Structure
```Tetris-End-To-End-Project/
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── package.json
│   ├── src/
│   └── public/
│
├── backend/
│   ├── Dockerfile
│   ├── server.js
│   └── package.json
│
└── README.md

```
## 🚀 Run Everything Locally (Step by Step)
### 1️⃣ Clone Repository
```git clone https://github.com/shymaasaed/Tetris-End-To-End-Project
cd tetris-project

```
### 2️⃣ Create Docker Network
All containers must be on the same network:
```docker network create tetris-net

```
### 3️⃣ Run Redis
```docker run -d \
--name redis \
--network tetris-net \
redis

```
Verify:
```docker ps

```
You should see redis running.
### 4️⃣ Build Frontend Image
```cd frontend
docker build -t tetris-frontend .

```
### 5️⃣ Run Frontend Container
```docker run -d \
--name frontend \
--network tetris-net \
-p 3000:80 \
tetris-frontend

```
Frontend will be available at:
👉 http://localhost:3000
### 6️⃣ Build Backend Image
Open a new terminal:
```cd backend
docker build -t tetris-backend .

```
### 7️⃣ Run Backend Container
```docker run -d \
--name backend \
--network tetris-net \
-p 4000:4000 \
-e REDIS_HOST=redis \
tetris-backend

```
Backend API will be available at:
👉 http://localhost:4000
Important:
- REDIS_HOST=redis must match Redis container name.
## 🧪 Backend Testing
#### Add score:
```curl -X POST http://localhost:4000/score \
-H "Content-Type: application/json" \
-d '{"playerId":"test","score":300}'

```
#### Get leaderboard:
```curl http://localhost:4000/leaderboard

```
You should receive leaderboard data.
## 🎮 Game Testing
Open browser:
👉 http://localhost:3000
Play the game.
After game over, score is automatically saved to Redis.
## 🚫 Important Notes
- Do NOT commit node_modules
- Do NOT commit .env files
- Redis hostname is provided using environment variables
- Frontend + Backend communicate through Docker network
## 🧹 Stop & Clean Everything
```docker rm -f frontend backend redis
docker network rm tetris-net

```
## ✅ Summary
This setup provides:
- React frontend in Nginx container
- Node 14 backend API container
- Redis container
- Docker network communication
- Persistent leaderboard
This mirrors real production architecture and prepares the project for AWS / EKS deployment.