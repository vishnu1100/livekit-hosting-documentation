
# LiveKit Self-Hosting Guide: EC2 + Docker + GitHub Pages Frontend

This guide helps you self-host the LiveKit server on an AWS EC2 instance using Docker, and deploy the Meet demo frontend on GitHub Pages using your own LiveKit server.

---

## Prerequisites

- AWS account with an Ubuntu 22.04 EC2 instance
- Docker & Docker Compose installed on EC2
- GitHub account for frontend hosting
- Basic Git and terminal knowledge

---

## 1. Setup LiveKit Server on EC2

### 1.1 Connect to EC2 via SSH

```bash
ssh -i your-key.pem ubuntu@your-ec2-public-ip
```

---

### 1.2 Install Docker and Docker Compose

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable docker --now
sudo usermod -aG docker $USER
newgrp docker
```

Verify:

```bash
docker --version
docker-compose --version
```

---

### 1.3 Create LiveKit Config File `livekit.yaml`

```yaml
keys:
  - key: your_api_key
    secret: your_api_secret

rtc:
  port: 7882
  udp_port_range: 7882-7883

http:
  port: 7880

signal:
  port: 7881
```

> Replace `your_api_key` and `your_api_secret` with strong secrets.

---

### 1.4 Create `docker-compose.yml`

```yaml
version: "3.7"

services:
  livekit:
    image: livekit/livekit-server:v1.6.3
    ports:
      - "7880:7880"
      - "7881:7881"
      - "7882:7882/udp"
    volumes:
      - ./livekit.yaml:/etc/livekit.yaml
    restart: always
```

---

### 1.5 Start LiveKit Server

```bash
docker-compose up -d
```

Check logs with:

```bash
docker-compose logs -f
```

---

## 2. Deploy Meet Frontend on GitHub Pages

### 2.1 Clone Meet Repo Locally

```bash
git clone https://github.com/livekit/meet.git
cd meet
```

---

### 2.2 Create `.env.local` file

```env
VITE_API_KEY=your_api_key
VITE_API_SECRET=your_api_secret
VITE_LIVEKIT_URL=wss://your-ec2-public-ip:7880
```

Replace placeholders accordingly.

---

### 2.3 Install dependencies and build frontend

```bash
npm install
npm run build
```

---

### 2.4 Push to GitHub and enable Pages

- Create a new GitHub repo (e.g., `livekit-meet-frontend`)
- Initialize git and push:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/yourusername/livekit-meet-frontend.git
git push -u origin main
```

- In GitHub repo settings, go to **Pages**
- Select branch: `main`, folder: `/ (root)`
- Save and wait for deployment

---

## 3. Usage

- Open the URL `https://yourusername.github.io/livekit-meet-frontend/`
- Join or create rooms connected to your LiveKit server running on EC2

---

## 4. Notes & Security

- **Security Warning:** Exposing your API Key and Secret in frontend code is not safe for public use.
- Use this setup only in private or trusted environments.
- For production, consider adding a backend token service for secure token generation.
- Consider setting up HTTPS with NGINX and certbot on EC2.

---

## 5. Summary of Commands

### On EC2:

```bash
ssh -i your-key.pem ubuntu@your-ec2-public-ip

sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable docker --now
sudo usermod -aG docker $USER
newgrp docker

# Create livekit.yaml and docker-compose.yml (see above)

docker-compose up -d
docker-compose logs -f
```

---

### Locally for frontend:

```bash
git clone https://github.com/livekit/meet.git
cd meet

# create .env.local with your API key/secret and server URL

npm install
npm run build

git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/yourusername/livekit-meet-frontend.git
git push -u origin main
```

---

Feel free to open issues or ask for help improving security or adding features!

---

**Enjoy your self-hosted LiveKit video conferencing!**
