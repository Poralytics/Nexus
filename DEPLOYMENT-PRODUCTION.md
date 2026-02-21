# 🚀 NEXUS PRODUCTION DEPLOYMENT GUIDE

## Guide Complet de Déploiement en Production

---

## 📋 Pre-Deployment Checklist

### Security
- [ ] Change tous les secrets par défaut
- [ ] JWT_SECRET généré aléatoirement (min 64 caractères)
- [ ] Admin password changé
- [ ] CORS configuré correctement (pas de `*` en production)
- [ ] Rate limiting activé
- [ ] Helmet.js activé
- [ ] SSL/TLS certificates configurés
- [ ] API keys en environment variables (jamais hardcodé)

### Database
- [ ] PostgreSQL configuré (pas SQLite en production)
- [ ] Backups automatiques activés
- [ ] Migrations exécutées
- [ ] Indexes créés
- [ ] Connection pooling configuré

### Infrastructure
- [ ] Redis installé et configuré
- [ ] Load balancer configuré
- [ ] Auto-scaling configuré
- [ ] Health checks configurés
- [ ] Monitoring activé
- [ ] Logging centralisé
- [ ] CDN configuré

### Environment
- [ ] NODE_ENV=production
- [ ] Toutes les variables d'environnement configurées
- [ ] Secrets stockés de manière sécurisée
- [ ] Backups des configurations

---

## 🌐 Option 1: Heroku (Le Plus Simple)

### Étape 1: Préparation

```bash
# Install Heroku CLI
brew install heroku  # Mac
# or
curl https://cli-assets.heroku.com/install.sh | sh  # Linux

# Login
heroku login
```

### Étape 2: Création App

```bash
# Create app
heroku create nexus-security-prod

# Add PostgreSQL
heroku addons:create heroku-postgresql:standard-0

# Add Redis
heroku addons:create heroku-redis:premium-0

# Get database URL
heroku config:get DATABASE_URL
```

### Étape 3: Configuration

```bash
# Set environment variables
heroku config:set NODE_ENV=production
heroku config:set JWT_SECRET=$(openssl rand -hex 32)
heroku config:set STRIPE_SECRET_KEY=sk_live_your_key
heroku config:set OPENAI_API_KEY=sk-your_key

# Set app URL
heroku config:set APP_URL=https://nexus-security-prod.herokuapp.com
```

### Étape 4: Deploy

```bash
# Add Heroku remote
git remote add heroku https://git.heroku.com/nexus-security-prod.git

# Deploy
git push heroku main

# Run migrations
heroku run npm run migrate

# Check logs
heroku logs --tail

# Open app
heroku open
```

### Étape 5: Scaling

```bash
# Scale dynos
heroku ps:scale web=2 worker=1

# Auto-scaling (add-on)
heroku addons:create scheduler:standard
```

**Coût Heroku:**
- Standard DB: $50/mois
- Premium Redis: $60/mois
- 2 web dynos: $50/mois
- **Total: ~$160/mois**

---

## 🚄 Option 2: Railway (Moderne & Rapide)

### Étape 1: Setup

1. Aller sur [railway.app](https://railway.app)
2. Connecter GitHub
3. "New Project" → "Deploy from GitHub"
4. Sélectionner ton repo

### Étape 2: Services

```bash
# Add PostgreSQL
Railway Dashboard → New → Database → PostgreSQL

# Add Redis
Railway Dashboard → New → Database → Redis

# Variables automatically set:
# - DATABASE_URL
# - REDIS_URL
```

### Étape 3: Variables

Dans Railway Dashboard → Variables:
```env
NODE_ENV=production
JWT_SECRET=your_secret_here
STRIPE_SECRET_KEY=sk_live_...
OPENAI_API_KEY=sk-...
PORT=3000
```

### Étape 4: Deploy

Railway auto-deploy sur chaque push Git.

```bash
git push origin main
# Railway détecte et déploie automatiquement
```

### Étape 5: Custom Domain

Railway Dashboard → Settings → Domains → Add Custom Domain

**Coût Railway:**
- $5/mois base
- ~$20/mois PostgreSQL
- ~$10/mois Redis
- **Total: ~$35/mois**

---

## 🏗️ Option 3: DigitalOcean (Plus de Contrôle)

### Étape 1: Créer Droplet

```bash
# Via web interface:
- Ubuntu 22.04 LTS
- 2 GB RAM / 1 vCPU ($12/mois)
- ou 4 GB RAM / 2 vCPU ($24/mois) recommandé

# SSH into server
ssh root@your_server_ip
```

### Étape 2: Setup Serveur

```bash
# Update system
apt update && apt upgrade -y

# Install Node.js 18
curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
apt install -y nodejs

# Install PostgreSQL
apt install -y postgresql postgresql-contrib

# Install Redis
apt install -y redis-server

# Install Nginx
apt install -y nginx

# Install PM2
npm install -g pm2

# Install certbot for SSL
apt install -y certbot python3-certbot-nginx
```

### Étape 3: Setup PostgreSQL

```bash
# Switch to postgres user
sudo -u postgres psql

# Create database and user
CREATE DATABASE nexus;
CREATE USER nexus_user WITH PASSWORD 'secure_password_here';
GRANT ALL PRIVILEGES ON DATABASE nexus TO nexus_user;
\q
```

### Étape 4: Setup Application

```bash
# Create app directory
mkdir -p /var/www/nexus
cd /var/www/nexus

# Clone repository
git clone your_repo.git .

# Install dependencies
cd backend
npm install --production

# Create .env
nano .env
# Add all production variables

# Run migrations
npm run migrate

# Start with PM2
pm2 start server.js --name nexus
pm2 save
pm2 startup
```

### Étape 5: Setup Nginx

```bash
# Create Nginx config
nano /etc/nginx/sites-available/nexus

# Add:
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

# Enable site
ln -s /etc/nginx/sites-available/nexus /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx
```

### Étape 6: Setup SSL

```bash
# Get SSL certificate
certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Auto-renewal test
certbot renew --dry-run
```

### Étape 7: Setup Backups

```bash
# Create backup script
nano /root/backup-nexus.sh

#!/bin/bash
cd /var/www/nexus/backend
node scripts/backup-database.js

# Make executable
chmod +x /root/backup-nexus.sh

# Add to crontab (daily at 2 AM)
crontab -e
0 2 * * * /root/backup-nexus.sh
```

### Étape 8: Monitoring

```bash
# PM2 monitoring
pm2 monit

# View logs
pm2 logs nexus

# Setup PM2 web dashboard
pm2 install pm2-server-monit
```

**Coût DigitalOcean:**
- Droplet 4GB: $24/mois
- Managed PostgreSQL: $15/mois
- Managed Redis: $15/mois
- Backups: $4.80/mois
- **Total: ~$59/mois**

---

## ☁️ Option 4: AWS (Enterprise Scale)

### Architecture

```
Internet → CloudFront (CDN)
          ↓
       ALB (Load Balancer)
          ↓
       ECS (Containers)
          ↓
    RDS PostgreSQL
    ElastiCache Redis
    S3 (File Storage)
```

### Étape 1: RDS PostgreSQL

```bash
# Via AWS Console:
- RDS → Create Database
- PostgreSQL 15
- db.t3.medium (2 vCPU, 4GB RAM)
- Multi-AZ for high availability
- Automated backups
```

### Étape 2: ElastiCache Redis

```bash
# Via AWS Console:
- ElastiCache → Create
- Redis 7.x
- cache.t3.medium
- Multi-AZ with automatic failover
```

### Étape 3: ECS/Fargate

```bash
# Create Dockerfile if not exists
# Create task definition
# Create ECS cluster
# Create service with auto-scaling
```

### Étape 4: CloudFront

```bash
# Create distribution
# Point to ALB
# Configure SSL certificate (ACM)
# Setup caching rules
```

### Étape 5: S3 for Storage

```bash
# Create bucket for reports/uploads
aws s3 mb s3://nexus-reports-prod
aws s3 mb s3://nexus-backups-prod
```

**Coût AWS** (estimation):
- ECS Fargate: ~$50/mois
- RDS db.t3.medium: ~$100/mois
- ElastiCache: ~$50/mois
- S3: ~$10/mois
- CloudFront: ~$20/mois
- ALB: ~$20/mois
- **Total: ~$250/mois**

---

## 🔐 Security Hardening

### 1. Firewall

```bash
# DigitalOcean/Ubuntu
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
```

### 2. Fail2ban

```bash
apt install -y fail2ban
systemctl enable fail2ban
```

### 3. Rate Limiting

Déjà configuré dans le code:
```javascript
// express-rate-limit already configured
// 100 requests per 15 minutes
```

### 4. Security Headers

Helmet.js déjà activé dans le code.

### 5. Database Security

```sql
-- Restrict database user permissions
REVOKE ALL ON DATABASE nexus FROM PUBLIC;
GRANT CONNECT ON DATABASE nexus TO nexus_user;
```

---

## 📊 Monitoring Setup

### Option 1: Sentry (Error Tracking)

```bash
# Install
npm install @sentry/node

# Configure
SENTRY_DSN=https://your_key@sentry.io/project
```

### Option 2: DataDog (APM)

```bash
# Install
npm install dd-trace --save

# In server.js (first line):
require('dd-trace').init()
```

### Option 3: LogTail (Logs)

```bash
# Configure
LOGTAIL_SOURCE_TOKEN=your_token
```

---

## 🚨 Emergency Procedures

### Rollback Deployment

```bash
# Heroku
heroku rollback

# Railway
Railway Dashboard → Deployments → Rollback

# DigitalOcean
pm2 stop nexus
git checkout previous_commit
npm install
npm run migrate
pm2 restart nexus
```

### Database Restore

```bash
# Stop app
pm2 stop nexus

# Restore from backup
cd /var/www/nexus/backend/backups
gunzip -c nexus-backup-TIMESTAMP.db.gz > ../nexus-ultimate.db

# Restart
pm2 restart nexus
```

### Emergency Contacts

Add to your playbook:
- Database admin
- DevOps lead
- On-call engineer
- Stripe support
- AWS support

---

## ✅ Post-Deployment Checklist

- [ ] Health check endpoint responding
- [ ] Database migrations successful
- [ ] Redis connection working
- [ ] Stripe webhooks receiving events
- [ ] Email notifications working
- [ ] SSL certificate valid
- [ ] Custom domain resolving
- [ ] Monitoring alerts configured
- [ ] Backup script running
- [ ] Load testing completed
- [ ] Security scan passed
- [ ] Documentation updated

---

## 📈 Scaling Strategy

### When to Scale

- CPU > 70% sustained
- Memory > 80% sustained
- Response time > 500ms
- Error rate > 1%

### How to Scale

**Horizontal (More Servers):**
```bash
# Heroku
heroku ps:scale web=4

# AWS ECS
Update desired count in service

# DigitalOcean
Add more droplets + load balancer
```

**Vertical (Bigger Servers):**
```bash
# Heroku
heroku ps:resize web=performance-l

# AWS
Change instance type

# DigitalOcean
Resize droplet
```

---

## 💰 Cost Optimization

### Development/Staging
- Heroku: Free tier + Hobby DB ($7/mois)
- Railway: $5/mois
- DigitalOcean: $6/mois droplet

### Small Production (< 1K users)
- Railway: ~$35/mois
- DigitalOcean: ~$30/mois
- **Recommandé: Railway**

### Medium Production (1K-10K users)
- DigitalOcean: ~$60/mois
- **Recommandé: DigitalOcean**

### Large Production (10K+ users)
- AWS: ~$250/mois base
- Auto-scaling activé
- **Recommandé: AWS**

---

## 🎯 Performance Targets

- Response time: < 200ms (p95)
- Uptime: > 99.9%
- Error rate: < 0.1%
- Database queries: < 100ms
- API throughput: 1000+ req/sec

---

## 📞 Support

Si problèmes en production:
1. Check logs: `pm2 logs` ou Heroku logs
2. Check monitoring: Sentry/DataDog
3. Check database: Connection OK?
4. Check Redis: Connection OK?
5. Rollback if critical

---

**NEXUS est maintenant PRODUCTION-READY!** 🚀

Choose your deployment platform and GO! 💎
