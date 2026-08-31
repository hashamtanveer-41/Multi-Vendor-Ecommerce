# Deployment Guide: Multi-Vendor E-Commerce Platform

This guide outlines deployment options for the **Multi-Vendor E-Commerce Platform**, covering containerized deployment (Docker), modern cloud platforms (Render + Vercel + MongoDB Atlas), and self-hosted Linux VPS setups (Nginx + PM2 + SSL).

---

## 📋 Table of Contents
1. [Prerequisites & Environment Variables](#1-prerequisites--environment-variables)
2. [Option 1: 🐳 Docker & Docker Compose (Recommended for Self-Hosting & VPS)](#option-1--docker--docker-compose)
3. [Option 2: ☁️ Cloud PaaS Deployment (Render + Vercel + MongoDB Atlas)](#option-2-️-cloud-paas-deployment-render--vercel--mongodb-atlas)
4. [Option 3: 🚀 Railway Full-Stack Deployment](#option-3--railway-full-stack-deployment)
5. [Option 4: 🖥️ Linux VPS Deployment (Nginx + PM2 + Let's Encrypt SSL)](#option-4-️-linux-vps-deployment-nginx--pm2--lets-encrypt-ssl)
6. [Post-Deployment Verification & Troubleshooting](#post-deployment-verification--troubleshooting)

---

## 1. Prerequisites & Environment Variables

### Environment Variables Reference

#### Backend Service (`backend/config/.env` or Hosting Dashboard)
| Variable | Description | Example / Default |
|---|---|---|
| `NODE_ENV` | Environment mode | `production` |
| `PORT` | Backend server port | `8000` |
| `FRONTEND_URL` | Allowed frontend origin(s), comma-separated | `https://your-frontend.vercel.app` |
| `DB_URL` | MongoDB Connection URI (Atlas or local) | `mongodb+srv://user:pass@cluster.mongodb.net/eshop` |
| `JWT_SECRET_KEY` | Secret key for signing JWT user tokens | `<random_64_char_string>` |
| `JWT_EXPIRES` | Expiration time for JWT tokens | `7d` |
| `ACTIVATION_SECRET` | Secret key for email verification tokens | `<random_64_char_string>` |
| `STRIPE_API_KEY` | Stripe Publishable Key | `pk_live_...` or `pk_test_...` |
| `STRIPE_SECRET_KEY` | Stripe Secret API Key | `sk_live_...` or `sk_test_...` |
| `CLOUDINARY_NAME` | Cloudinary Cloud Name | `your_cloudinary_name` |
| `CLOUDINARY_API_KEY` | Cloudinary API Key | `your_cloudinary_api_key` |
| `CLOUDINARY_API_SECRET` | Cloudinary API Secret | `your_cloudinary_api_secret` |
| `SMPT_SERVICE` | Mail provider service | `gmail` |
| `SMPT_HOST` | SMTP server host | `smtp.gmail.com` |
| `SMPT_PORT` | SMTP port | `465` (SSL) or `587` (TLS) |
| `SMPT_MAIL` | Sender email address | `noreply@yourdomain.com` |
| `SMPT_PASSWORD` | App password (e.g. Gmail App Password) | `<your_app_password>` |

#### Socket Service (`socket/.env` or Hosting Dashboard)
| Variable | Description | Example / Default |
|---|---|---|
| `PORT` | Socket server port | `4000` |
| `FRONTEND_URL` | Allowed client origin(s), comma-separated | `https://your-frontend.vercel.app` |

#### Frontend Service (`frontend/.env` or Hosting Dashboard)
| Variable | Description | Example / Default |
|---|---|---|
| `REACT_APP_SERVER_URL` | REST API base endpoint | `https://your-api.onrender.com/api/v2` |
| `REACT_APP_BACKEND_URL` | Backend server root endpoint | `https://your-api.onrender.com/` |
| `REACT_APP_SOCKET_URL` | Socket.IO server endpoint | `https://your-socket.onrender.com/` |

---

## Option 1: 🐳 Docker & Docker Compose

Docker Compose orchestrates the entire application including MongoDB, Backend, Socket, and Frontend (Nginx) in isolated containers with persistent volumes.

### Quick Start:

1. **Clone repository & navigate to project root**:
   ```bash
   git clone https://github.com/hashamtanveer-41/Multi-Vendor-Ecommerce.git
   cd Multi-Vendor-Ecommerce
   ```

2. **(Optional) Configure environment overrides**:
   Create a root `.env` file or export your secrets:
   ```bash
   export JWT_SECRET_KEY="your_secure_random_key"
   export ACTIVATION_SECRET="your_activation_secret"
   export CLOUDINARY_NAME="your_cloudinary_name"
   export CLOUDINARY_API_KEY="your_cloudinary_api_key"
   export CLOUDINARY_API_SECRET="your_cloudinary_api_secret"
   export STRIPE_API_KEY="pk_test_..."
   export STRIPE_SECRET_KEY="sk_test_..."
   ```

3. **Build and launch containers**:
   ```bash
   docker compose up -d --build
   ```

4. **Access the application**:
   - Frontend: `http://localhost:3000`
   - Backend REST API: `http://localhost:8000`
   - Socket Server: `http://localhost:4000`
   - MongoDB: `mongodb://localhost:27017`

5. **Useful Docker management commands**:
   ```bash
   # View container logs
   docker compose logs -f

   # Stop all services
   docker compose down

   # Restart a specific service
   docker compose restart backend
   ```

---

## Option 2: ☁️ Cloud PaaS Deployment (Render + Vercel + MongoDB Atlas)

This combination offers a reliable free/low-cost cloud architecture.

### Step A: Set up Database (MongoDB Atlas)
1. Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) and create a free M0 cluster.
2. Under **Database Access**, create a database user and password.
3. Under **Network Access**, add IP `0.0.0.0/0` (allow access from anywhere).
4. Click **Connect** > **Drivers** and copy your connection string (e.g. `mongodb+srv://<user>:<password>@cluster0.abcde.mongodb.net/eshop?retryWrites=true&w=majority`).

### Step B: Deploy Backend & Socket to Render
1. Push your repository to GitHub.
2. Log into [Render](https://render.com).
3. **Deploy via Render Blueprint (`render.yaml`)**:
   - Click **New +** > **Blueprint**.
   - Connect your GitHub repository.
   - Render will detect `render.yaml` and configure:
     - `eshop-backend` (Node Web Service)
     - `eshop-socket` (Node Web Service)
   - In the Render Dashboard, fill in the secret environment variables (`DB_URL`, `CLOUDINARY_*`, `STRIPE_*`, `SMPT_*`).
4. Note your deployed Backend URL (e.g. `https://eshop-backend.onrender.com`) and Socket URL (e.g. `https://eshop-socket.onrender.com`).

### Step C: Deploy Frontend to Vercel
1. Go to [Vercel](https://vercel.com) and click **Add New...** > **Project**.
2. Import your GitHub repository.
3. In **Root Directory**, choose `frontend`.
4. In **Build Settings**:
   - Build Command: `npm install --legacy-peer-deps && npm run build`
   - Output Directory: `build`
   - Install Command: `npm install --legacy-peer-deps`
5. In **Environment Variables**, add:
   - `REACT_APP_SERVER_URL` = `https://eshop-backend.onrender.com/api/v2`
   - `REACT_APP_BACKEND_URL` = `https://eshop-backend.onrender.com/`
   - `REACT_APP_SOCKET_URL` = `https://eshop-socket.onrender.com/`
6. Click **Deploy**.
7. Once deployed, copy your Vercel domain (e.g. `https://eshop-frontend.vercel.app`).

### Step D: Update `FRONTEND_URL` on Backend & Socket
1. Go back to Render Dashboard.
2. In `eshop-backend` and `eshop-socket` Environment tabs, set:
   - `FRONTEND_URL` = `https://eshop-frontend.vercel.app`
3. Trigger a manual redeploy on both services.

---

## Option 3: 🚀 Railway Full-Stack Deployment

1. Go to [Railway](https://railway.app) and create a **New Project**.
2. Click **Add a Service** > **Database** > **MongoDB**.
3. Click **Add a Service** > **GitHub Repo** > Select this repository:
   - In Service Settings, set **Root Directory** to `/backend`.
   - Add environment variables referencing `MONGO_URL` and your credentials.
   - Generate a Railway domain for public networking.
4. Click **Add a Service** > **GitHub Repo** > Select this repository:
   - In Service Settings, set **Root Directory** to `/socket`.
   - Add `PORT` (e.g. `4000` or `$PORT`) and `FRONTEND_URL`.
   - Generate a Railway domain for the socket service.
5. Click **Add a Service** > **GitHub Repo** > Select this repository:
   - In Service Settings, set **Root Directory** to `/frontend`.
   - Add `REACT_APP_SERVER_URL`, `REACT_APP_BACKEND_URL`, and `REACT_APP_SOCKET_URL`.

---

## Option 4: 🖥️ Linux VPS Deployment (Nginx + PM2 + Let's Encrypt SSL)

For deploying onto Ubuntu/Debian on AWS EC2, DigitalOcean Droplet, Linode, or Hetzner.

### 1. Server Setup
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Node.js 18 LTS & Nginx
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs nginx git certbot python3-certbot-nginx

# Install PM2 globally
sudo npm install -g pm2
```

### 2. Clone & Build
```bash
cd /var/www
sudo git clone https://github.com/hashamtanveer-41/Multi-Vendor-Ecommerce.git eshop
cd eshop

# Setup backend
cd /var/www/eshop/backend
npm install
cp config/.env.example config/.env
# Edit config/.env with production credentials: nano config/.env

# Setup socket
cd /var/www/eshop/socket
npm install
cp .env.example .env
# Edit .env: nano .env

# Setup and build frontend
cd /var/www/eshop/frontend
npm install --legacy-peer-deps
cat << 'EOF' > .env
REACT_APP_SERVER_URL=https://api.yourdomain.com/api/v2
REACT_APP_BACKEND_URL=https://api.yourdomain.com/
REACT_APP_SOCKET_URL=https://socket.yourdomain.com/
