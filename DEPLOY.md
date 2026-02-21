# 🚀 NEXUS ULTIMATE PRO - Guide de Déploiement

## Déploiement avec Docker (Recommandé)

### Prérequis
- Docker 20.10+
- Docker Compose 2.0+
- 2GB RAM minimum
- 5GB espace disque

### Installation Rapide

```bash
# 1. Cloner/Extraire le projet
tar -xzf NEXUS-ULTIMATE-PRO-FINAL.tar.gz
cd NEXUS-FINAL-PRO

# 2. Configurer l'environnement
cd docker
cp .env.example .env
nano .env  # Éditer les variables

# 3. Lancer tous les services
docker-compose up -d

# 4. Vérifier les logs
docker-compose logs -f

# 5. Initialiser la base de données
docker-compose exec backend node init-nexus.js

# 6. Accéder à l'application
# Frontend: http://localhost
# API: http://localhost/api
# Backend direct: http://localhost:3000
```

### Configuration .env

**IMPORTANT** - Changez ces valeurs :

```bash
# Database (SÉCURISEZ CES MOTS DE PASSE)
DB_PASSWORD=VotreMotDePasseSecure123!@#

# JWT (Générez une clé aléatoire de 64 caractères)
JWT_SECRET=$(openssl rand -base64 64)

# Email (pour notifications)
SMTP_USER=votre-email@gmail.com
SMTP_PASS=votre-mot-de-passe-application

# Intégrations (optionnel)
SLACK_WEBHOOK_URL=https://hooks.slack.com/...
JIRA_API_TOKEN=votre-token
GITHUB_TOKEN=ghp_votre_token
```

### Commandes Docker Utiles

```bash
# Arrêter tous les services
docker-compose down

# Redémarrer
docker-compose restart

# Voir les logs
docker-compose logs -f backend
docker-compose logs -f worker

# Accéder au conteneur
docker-compose exec backend sh
docker-compose exec postgres psql -U nexus nexus_pro

# Nettoyer les volumes (ATTENTION: supprime les données)
docker-compose down -v
```

---

## Déploiement Manuel (Sans Docker)

### Prérequis
- Node.js 18+
- PostgreSQL 15+
- Redis 7+
- Nginx

### Installation PostgreSQL

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install postgresql postgresql-contrib

# Créer base de données
sudo -u postgres psql
CREATE DATABASE nexus_pro;
CREATE USER nexus WITH ENCRYPTED PASSWORD 'votre_password';
GRANT ALL PRIVILEGES ON DATABASE nexus_pro TO nexus;
\q
```

### Installation Redis

```bash
# Ubuntu/Debian
sudo apt install redis-server
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

### Installation NEXUS

```bash
# 1. Extraire
tar -xzf NEXUS-ULTIMATE-PRO-FINAL.tar.gz
cd NEXUS-FINAL-PRO/backend

# 2. Installer dépendances
npm install --production

# 3. Configurer environnement
cp .env.example .env
nano .env  # Éditer avec vos valeurs

# 4. Initialiser DB
node init-nexus.js

# 5. Lancer backend
npm start

# 6. Lancer worker (nouveau terminal)
node workers/scan-worker.js
```

### Configuration Nginx

```nginx
# /etc/nginx/sites-available/nexus

server {
    listen 80;
    server_name votre-domaine.com;

    # Frontend
    location / {
        root /path/to/NEXUS-FINAL-PRO/frontend;
        try_files $uri $uri/ /index.html;
    }

    # API
    location /api/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Activer le site
sudo ln -s /etc/nginx/sites-available/nexus /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Service Systemd (Auto-restart)

```ini
# /etc/systemd/system/nexus-backend.service

[Unit]
Description=NEXUS Backend
After=network.target postgresql.service redis.service

[Service]
Type=simple
User=www-data
WorkingDirectory=/path/to/NEXUS-FINAL-PRO/backend
ExecStart=/usr/bin/node server.js
Restart=always
RestartSec=10
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

```bash
# Activer et démarrer
sudo systemctl enable nexus-backend
sudo systemctl start nexus-backend
sudo systemctl status nexus-backend
```

---

## Déploiement Cloud

### AWS

**EC2 + RDS + ElastiCache**

```bash
# 1. Lancer EC2 (t3.medium minimum)
# 2. Créer RDS PostgreSQL
# 3. Créer ElastiCache Redis
# 4. Configurer Security Groups
# 5. SSH vers EC2 et installer NEXUS
# 6. Pointer .env vers RDS et ElastiCache
```

### Google Cloud

**Compute Engine + Cloud SQL + Memorystore**

```bash
# 1. Créer VM Compute Engine
# 2. Créer Cloud SQL PostgreSQL
# 3. Créer Memorystore Redis
# 4. Installer NEXUS sur VM
# 5. Configurer connexions privées
```

### Azure

**VM + Azure Database + Azure Cache**

```bash
# 1. Créer Azure VM
# 2. Créer Azure Database for PostgreSQL
# 3. Créer Azure Cache for Redis
# 4. Déployer NEXUS
# 5. Configurer networking
```

### Heroku (Rapide)

```bash
# 1. Installer Heroku CLI
heroku login

# 2. Créer app
heroku create nexus-security

# 3. Ajouter add-ons
heroku addons:create heroku-postgresql:hobby-dev
heroku addons:create heroku-redis:hobby-dev

# 4. Déployer
git init
git add .
git commit -m "Deploy NEXUS"
git push heroku main

# 5. Initialiser DB
heroku run node init-nexus.js
```

---

## SSL/HTTPS (Let's Encrypt)

### Avec Certbot

```bash
# 1. Installer Certbot
sudo apt install certbot python3-certbot-nginx

# 2. Obtenir certificat
sudo certbot --nginx -d votre-domaine.com

# 3. Auto-renouvellement
sudo certbot renew --dry-run
```

### Avec Docker

```yaml
# Ajouter dans docker-compose.yml

  certbot:
    image: certbot/certbot
    volumes:
      - ./ssl:/etc/letsencrypt
    command: certonly --webroot --webroot-path=/var/www/certbot --email your@email.com --agree-tos --no-eff-email -d votre-domaine.com
```

---

## Monitoring

### Logs

```bash
# Backend logs
tail -f /var/log/nexus/backend.log

# Nginx logs
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# Docker logs
docker-compose logs -f --tail=100
```

### Prometheus + Grafana

```yaml
# Ajouter dans docker-compose.yml

  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
```

---

## Backup

### Base de données

```bash
# Backup PostgreSQL
pg_dump -U nexus nexus_pro > backup-$(date +%Y%m%d).sql

# Restaurer
psql -U nexus nexus_pro < backup-20240214.sql

# Automatiser (crontab)
0 2 * * * /usr/bin/pg_dump -U nexus nexus_pro > /backups/nexus-$(date +\%Y\%m\%d).sql
```

### Fichiers

```bash
# Backup rapports
tar -czf reports-backup-$(date +%Y%m%d).tar.gz /app/reports/

# Backup complet
tar -czf nexus-full-backup-$(date +%Y%m%d).tar.gz \
  /path/to/NEXUS-FINAL-PRO/ \
  /etc/nginx/sites-available/nexus \
  /etc/systemd/system/nexus-*.service
```

---

## Troubleshooting

### Backend ne démarre pas

```bash
# Vérifier logs
docker-compose logs backend

# Vérifier connectivité DB
docker-compose exec backend node -e "require('./config/database')"

# Vérifier port
netstat -tulpn | grep 3000
```

### Scans ne fonctionnent pas

```bash
# Vérifier worker
docker-compose logs worker

# Vérifier Redis
docker-compose exec redis redis-cli ping

# Redémarrer worker
docker-compose restart worker
```

### Erreurs de permission

```bash
# Corriger permissions
sudo chown -R www-data:www-data /path/to/NEXUS-FINAL-PRO
sudo chmod -R 755 /path/to/NEXUS-FINAL-PRO
sudo chmod -R 777 /path/to/NEXUS-FINAL-PRO/backend/reports
```

---

## Scaling

### Horizontal Scaling

```yaml
# docker-compose.yml

services:
  worker:
    deploy:
      replicas: 5  # 5 workers en parallèle
    
  backend:
    deploy:
      replicas: 3  # 3 instances backend
```

### Load Balancing

```nginx
# nginx.conf

upstream backend {
    least_conn;
    server backend1:3000;
    server backend2:3000;
    server backend3:3000;
}
```

---

## Support

**Documentation**: README-FINAL.md  
**Issues**: Créer ticket GitHub  
**Email**: support@nexus-security.com  

---

**NEXUS ULTIMATE PRO - Production Deployment Guide**
