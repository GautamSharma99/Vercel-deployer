# Vercel Deployer

A microservices-based deployment platform similar to Vercel, built with Node.js, TypeScript, Redis, and AWS S3.

## Architecture

This project consists of three main services:

1. **vercel-upload-service** - Main API server that handles deployment requests (Port 3000)
2. **vercel-deploy-service** - Background worker that builds and deploys projects
3. **vercel-request-handler** - Handles serving deployed applications
4. **frontend** - React + TypeScript frontend (Port 5173)

## Prerequisites

Before running the backend services, make sure you have:

- **Node.js** (v18 or higher)
- **Redis** server running locally or remotely
- **AWS Account** with S3 configured (for file storage)
- **Git** installed

### Installing Redis on Windows

You have several options:

1. **Using WSL (Recommended)**:
   ```bash
   wsl --install
   # After WSL is installed, open Ubuntu and run:
   sudo apt update
   sudo apt install redis-server
   sudo service redis-server start
   ```

2. **Using Docker**:
   ```bash
   docker run -d -p 6379:6379 redis
   ```

3. **Using Memurai** (Redis alternative for Windows):
   - Download from: https://www.memurai.com/

## Setup Instructions

### 1. Install Dependencies

```bash
# Install frontend dependencies
cd frontend
npm install

# Install upload service dependencies
cd ../vercel-upload-service
npm install

# Install deploy service dependencies
cd ../vercel-deploy-service
npm install

# Install request handler dependencies (if needed)
cd ../vercel-request-handler
npm install
```

### 2. Configure Environment Variables

Copy the `.env.example` files to `.env` in each service directory and update with your credentials:

**vercel-upload-service**:
```bash
cd vercel-upload-service
cp .env.example .env
# Edit .env with your actual credentials
```

**vercel-deploy-service**:
```bash
cd vercel-deploy-service
cp .env.example .env
# Edit .env with your actual credentials
```

**Required Environment Variables**:
- `R2_ACCESS_KEY_ID` - Your Cloudflare R2 access key ID
- `R2_SECRET_ACCESS_KEY` - Your Cloudflare R2 secret access key
- `R2_ENDPOINT` - Your Cloudflare R2 endpoint URL
- `R2_BUCKET_NAME` - Your R2 bucket name (default: "vercel")
- `REDIS_URL` - Redis connection URL (default: "redis://localhost:6379")

> **Note**: The project uses **Cloudflare R2** for object storage, which is S3-compatible.

## Running the Services

### Start Redis (if using WSL)
```bash
wsl
sudo service redis-server start
```

### Start the Backend Services

Open **3 separate terminal windows**:

**Terminal 1 - Upload Service (Main API)**:
```bash
cd vercel-upload-service
npm run dev
```
This starts the API server on `http://localhost:3000`

**Terminal 2 - Deploy Service (Build Worker)**:
```bash
cd vercel-deploy-service
npm run dev
```
This starts the background worker that processes build queue

**Terminal 3 - Frontend**:
```bash
cd frontend
npm run dev
```
This starts the frontend on `http://localhost:5173`

## API Endpoints

### POST /deploy
Deploy a new project from a Git repository.

**Request**:
```json
{
  "repoUrl": "https://github.com/username/repo.git"
}
```

**Response**:
```json
{
  "id": "abc123"
}
```

### GET /status?id=abc123
Check deployment status.

**Response**:
```json
{
  "status": "uploaded" | "deployed"
}
```

## Troubleshooting

### Redis Connection Issues
- Make sure Redis is running: `redis-cli ping` (should return "PONG")
- Check if Redis is listening on port 6379: `netstat -an | findstr 6379`

### Cloudflare R2 Issues
- Verify your R2 credentials are correct in the `.env` file
- Ensure your R2 bucket exists and you have proper permissions
- Check the R2 endpoint URL matches your account
- Test connectivity: `curl https://your_account_id.r2.cloudflarestorage.com`

### Port Already in Use
If port 3000 or 5173 is already in use:
```bash
# Windows - Find and kill process using port
netstat -ano | findstr :3000
taskkill /PID <PID> /F
```

## Development

### Build for Production
```bash
# Build each service
cd vercel-upload-service && npm run build
cd ../vercel-deploy-service && npm run build
cd ../frontend && npm run build
```

### Run Production Build
```bash
# After building
npm start
```
