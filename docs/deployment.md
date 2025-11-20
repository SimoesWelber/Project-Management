# Deployment Guide

## INCA Deployment Documentation

This guide covers deploying the INCA platform to production environments.

## Prerequisites

### Infrastructure Requirements

**Minimum Requirements:**
- 2 CPU cores
- 4 GB RAM
- 50 GB storage
- PostgreSQL 13+ or MySQL 8+

**Recommended Requirements:**
- 4 CPU cores
- 8 GB RAM
- 100 GB SSD storage
- Load balancer for high availability
- Database replication

### Software Requirements

- Node.js v16+ (or Python 3.9+)
- Nginx or Apache (reverse proxy)
- PostgreSQL 13+ or MySQL 8+
- SSL certificate
- Git

## Deployment Options

### Option 1: Traditional Server Deployment

#### 1. Server Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install -y nodejs

# Install PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Install Nginx
sudo apt install -y nginx

# Install PM2 (process manager)
sudo npm install -g pm2
```

#### 2. Application Deployment

```bash
# Clone repository
cd /var/www
sudo git clone https://github.com/SimoesWelber/Project-Management.git inca
cd inca

# Install backend dependencies
cd backend
npm ci --production

# Install frontend dependencies and build
cd ../frontend
npm ci
npm run build

# Copy build to Nginx directory
sudo cp -r build/* /var/www/html/inca/
```

#### 3. Database Setup

```bash
# Create database
sudo -u postgres psql
CREATE DATABASE inca_prod;
CREATE USER inca_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE inca_prod TO inca_user;
\q

# Run migrations
cd /var/www/inca/backend
npm run migrate
```

#### 4. Configuration

```bash
# Create production .env file
cd /var/www/inca
sudo nano .env

# Add production values:
NODE_ENV=production
PORT=3000
DB_HOST=localhost
DB_PORT=5432
DB_NAME=inca_prod
DB_USER=inca_user
DB_PASSWORD=secure_password
JWT_SECRET=your_production_jwt_secret
# ... other variables
```

#### 5. Nginx Configuration

```bash
sudo nano /etc/nginx/sites-available/inca
```

```nginx
# Frontend
server {
    listen 80;
    server_name your-domain.com;
    
    # Redirect to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;
    
    ssl_certificate /etc/ssl/certs/your-cert.crt;
    ssl_certificate_key /etc/ssl/private/your-key.key;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Frontend
    location / {
        root /var/www/html/inca;
        try_files $uri $uri/ /index.html;
    }
    
    # API proxy
    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
    
    # File size limit
    client_max_body_size 10M;
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/inca /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

#### 6. Start Application with PM2

```bash
cd /var/www/inca/backend

# Start with PM2
pm2 start npm --name "inca-api" -- start

# Save PM2 configuration
pm2 save

# Set up PM2 to start on boot
pm2 startup
```

### Option 2: Docker Deployment

#### 1. Backend Dockerfile

```dockerfile
# backend/Dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --production

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

#### 2. Frontend Dockerfile

```dockerfile
# frontend/Dockerfile
FROM node:16-alpine as build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

#### 3. Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  database:
    image: postgres:13
    environment:
      POSTGRES_DB: inca_prod
      POSTGRES_USER: inca_user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - inca-network

  backend:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DB_HOST: database
      DB_PORT: 5432
      DB_NAME: inca_prod
      DB_USER: inca_user
      DB_PASSWORD: ${DB_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      - database
    networks:
      - inca-network

  frontend:
    build: ./frontend
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - backend
    networks:
      - inca-network

volumes:
  postgres_data:

networks:
  inca-network:
    driver: bridge
```

#### 4. Deploy with Docker

```bash
# Build and start
docker-compose up -d

# View logs
docker-compose logs -f

# Stop
docker-compose down
```

### Option 3: Cloud Deployment (AWS Example)

#### Architecture
- EC2 instances for application
- RDS for database
- S3 for file storage
- CloudFront for CDN
- Route 53 for DNS
- Load Balancer for high availability

#### Steps
1. Create RDS PostgreSQL instance
2. Create EC2 instances
3. Set up Auto Scaling Group
4. Configure Application Load Balancer
5. Create S3 bucket for uploads
6. Configure CloudFront distribution
7. Deploy application to EC2
8. Configure Route 53

## SSL/TLS Setup

### Using Let's Encrypt (Certbot)

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d your-domain.com

# Auto-renewal is configured automatically
# Test renewal
sudo certbot renew --dry-run
```

## Environment Variables

Production `.env` file should include:

```bash
NODE_ENV=production
PORT=3000
API_BASE_URL=https://your-domain.com

# Database
DB_HOST=your-db-host
DB_PORT=5432
DB_NAME=inca_prod
DB_USER=inca_user
DB_PASSWORD=secure_random_password

# Security
JWT_SECRET=secure_random_jwt_secret
SESSION_SECRET=secure_random_session_secret

# File Storage
UPLOAD_DIR=/var/inca/uploads
MAX_FILE_SIZE=10485760

# Email
EMAIL_HOST=smtp.your-provider.com
EMAIL_PORT=587
EMAIL_USER=your-email@domain.com
EMAIL_PASSWORD=your-email-password
```

## Database Backup

### Automated Backup Script

```bash
#!/bin/bash
# /usr/local/bin/backup-inca-db.sh

BACKUP_DIR="/var/backups/inca"
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_FILE="$BACKUP_DIR/inca_backup_$TIMESTAMP.sql"

# Create backup
pg_dump -h localhost -U inca_user inca_prod > $BACKUP_FILE

# Compress
gzip $BACKUP_FILE

# Remove backups older than 30 days
find $BACKUP_DIR -name "inca_backup_*.sql.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_FILE.gz"
```

### Schedule with Cron

```bash
# Edit crontab
sudo crontab -e

# Add daily backup at 2 AM
0 2 * * * /usr/local/bin/backup-inca-db.sh
```

## Monitoring

### PM2 Monitoring

```bash
# View status
pm2 status

# View logs
pm2 logs inca-api

# Monitor resources
pm2 monit
```

### System Monitoring

Install monitoring tools:
- Prometheus + Grafana
- New Relic
- Datadog
- AWS CloudWatch (for AWS)

## Security Hardening

### Firewall Configuration

```bash
# UFW (Ubuntu)
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw enable
```

### Fail2Ban Setup

```bash
# Install
sudo apt install fail2ban

# Configure
sudo nano /etc/fail2ban/jail.local

# Add:
[sshd]
enabled = true
maxretry = 3
bantime = 3600
```

## Health Checks

### API Health Endpoint

```javascript
// backend/src/routes/health.js
router.get('/health', async (req, res) => {
  try {
    // Check database connection
    await db.query('SELECT 1');
    
    res.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      uptime: process.uptime()
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      error: error.message
    });
  }
});
```

## Rollback Procedure

```bash
# If deployment fails:

# 1. Stop application
pm2 stop inca-api

# 2. Restore database backup
gunzip -c /var/backups/inca/inca_backup_TIMESTAMP.sql.gz | psql -U inca_user inca_prod

# 3. Revert code
cd /var/www/inca
git checkout previous-stable-tag

# 4. Reinstall dependencies
cd backend
npm ci --production

# 5. Restart application
pm2 restart inca-api
```

## Post-Deployment Checklist

- [ ] Application is accessible
- [ ] HTTPS is working
- [ ] Database connections working
- [ ] File uploads working
- [ ] Email notifications working
- [ ] Authentication working
- [ ] All API endpoints responding
- [ ] Logs are being written
- [ ] Backups are scheduled
- [ ] Monitoring is active
- [ ] SSL certificate auto-renewal configured
- [ ] Firewall rules configured
- [ ] Health checks passing

## Troubleshooting

### Application won't start
```bash
# Check logs
pm2 logs inca-api

# Check environment variables
pm2 env inca-api

# Check port availability
sudo netstat -tulpn | grep 3000
```

### Database connection issues
```bash
# Test connection
psql -h localhost -U inca_user -d inca_prod

# Check PostgreSQL status
sudo systemctl status postgresql
```

### Nginx issues
```bash
# Test configuration
sudo nginx -t

# Check logs
sudo tail -f /var/log/nginx/error.log
```

## Support

For deployment issues:
- Check logs first
- Review this guide
- Contact DevOps team
- Create a support ticket
