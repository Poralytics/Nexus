# ⚡ NEXUS QUICK START - 5 MINUTES TO LAUNCH

## 🚀 De Zéro à Lancé en 5 Minutes

---

## Méthode 1: Setup Automatique (RECOMMANDÉ)

```bash
# 1. Extract l'archive
tar -xzf NEXUS-v5.0-FINAL-100-PERCENT-COMPLETE.tar.gz
cd NEXUS-FINAL-PRO/backend

# 2. Run le setup automatique
node setup.js

# Suit les prompts interactifs:
# - Install dependencies? → y
# - Setup database? → y
# - Admin email: admin@example.com
# - Admin password: YourSecurePassword123
# - Redis installed? → n (skip pour maintenant)
# - Stripe? → n (skip pour maintenant)
# - OpenAI? → n (skip pour maintenant)

# 3. Start!
npm start

# 4. Open http://localhost:3000
```

**C'EST TOUT! NEXUS tourne.** ✅

---

## Méthode 2: Setup Manuel (5 étapes)

### Étape 1: Install (1 minute)
```bash
cd NEXUS-FINAL-PRO/backend
npm install
```

### Étape 2: Config (1 minute)
```bash
cp .env.example .env
nano .env

# Minimum requis:
NODE_ENV=development
PORT=3000
JWT_SECRET=change-this-super-secret-key-in-production
```

### Étape 3: Database (1 minute)
```bash
npm run migrate
```

### Étape 4: Admin User (1 minute)
```bash
npm run init
# Ou créer manuellement via setup.js
```

### Étape 5: Start (instant)
```bash
npm start
# Ou en dev mode:
npm run dev
```

**Open:** http://localhost:3000

---

## 🎯 Premier Login

1. Navigate to: http://localhost:3000
2. Click "Login"
3. Use admin credentials:
   - Email: admin@nexus.com (default)
   - Password: admin123 (CHANGE THIS!)

4. Go to dashboard
5. Add your first domain
6. Run your first scan

**You're live!** 🎉

---

## ⚙️ Configuration Optionnelle

### Pour Activer Toutes les Features

```bash
# Edit .env
nano .env
```

Add:
```env
# Stripe (pour billing)
STRIPE_SECRET_KEY=sk_test_your_key_here
STRIPE_PRICE_PRO=price_your_pro_price_id
STRIPE_PRICE_BUSINESS=price_your_business_price_id

# Redis (pour caching + queues)
REDIS_URL=redis://localhost:6379

# OpenAI (pour AI assistant)
OPENAI_API_KEY=sk-your_openai_key_here

# SMTP (pour emails)
SMTP_HOST=smtp.gmail.com
SMTP_USER=your_email@gmail.com
SMTP_PASSWORD=your_app_password
```

Restart:
```bash
npm start
```

---

## 🐳 Docker Quick Start (Alternative)

```bash
cd NEXUS-FINAL-PRO/docker
docker-compose up -d
```

Services lancés:
- Backend: http://localhost:3000
- PostgreSQL: localhost:5432
- Redis: localhost:6379

---

## ✅ Vérification

### Test 1: Health Check
```bash
curl http://localhost:3000/health
```

Should return:
```json
{
  "status": "OK",
  "timestamp": "...",
  "version": "2.0.0"
}
```

### Test 2: API Works
```bash
curl http://localhost:3000/api/auth/register \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"test123"}'
```

### Test 3: Frontend Loads
Open in browser: http://localhost:3000

Should see NEXUS landing page.

---

## 🔧 Troubleshooting

### "Cannot find module"
```bash
npm install
```

### "Port 3000 already in use"
```bash
# Find process
lsof -i :3000

# Kill it
kill -9 <PID>

# Or change port in .env
PORT=3001
```

### "Database locked"
```bash
# Stop all node processes
killall node

# Restart
npm start
```

### "Redis connection failed"
Features Redis-dependent (caching, queues) won't work.
Either:
1. Install Redis: `brew install redis && brew services start redis`
2. Or disable in .env: `ENABLE_REDIS=false`

---

## 📊 Next Steps

### Immediate (5 minutes)
1. ✅ Change admin password
2. ✅ Add your domain
3. ✅ Run first scan
4. ✅ View results

### Today (1 hour)
1. Configure Stripe for billing
2. Setup Redis for performance
3. Configure email notifications
4. Explore all features

### This Week
1. Configure AI assistant (OpenAI)
2. Deploy to staging
3. Invite team members
4. Run comprehensive scans

---

## 🎓 Learning Resources

### Guides in `/docs`:
- SETUP-GUIDE.md (detailed setup)
- DEPLOYMENT-PRODUCTION.md (deploy to prod)
- GO-TO-MARKET-STRATEGY.md (business strategy)
- STATUS-FINAL-100.md (complete features list)

### Routes API:
Check: http://localhost:3000/api/health

Full API docs will be at: /api/docs (Swagger - coming soon)

---

## 💡 Pro Tips

### Development
```bash
# Watch mode (auto-reload)
npm run dev

# Run tests
npm test

# Check code quality
npm run lint

# Format code
npm run format
```

### Production
```bash
# Always use production mode
NODE_ENV=production npm start

# Use PM2 for process management
pm2 start server.js --name nexus
pm2 save
pm2 startup
```

### Database
```bash
# Backup
node scripts/backup-database.js

# Verify installation
node scripts/verify-installation.js
```

---

## 🚀 Ready for Production?

### Checklist:
- [ ] Changed JWT_SECRET
- [ ] Changed admin password
- [ ] Configured all env variables
- [ ] Database backups setup
- [ ] SSL/HTTPS configured
- [ ] Monitoring setup
- [ ] Tests passing

### Deploy:
See DEPLOYMENT-PRODUCTION.md for detailed guides:
- Heroku (easiest)
- Railway (modern)
- DigitalOcean (control)
- AWS (enterprise)

---

## 📞 Need Help?

### Check First:
1. Logs: `npm start` output
2. Verify script: `node scripts/verify-installation.js`
3. Health check: `curl http://localhost:3000/health`

### Common Issues:
- Dependencies: Run `npm install`
- Database: Run `npm run migrate`
- Port conflict: Change PORT in .env
- Redis: Optional, can disable

---

## 🎉 You're Ready!

**NEXUS is running on your machine.**

**Features Available:**
✅ 20 Security Scanners
✅ Business Impact Analysis
✅ ML Anomaly Detection
✅ PDF/Excel Reports
✅ Real-time Scanning
✅ Vulnerability Database
✅ Analytics Dashboard

**Premium Features (Need Config):**
⚙️ Billing/Subscriptions (Stripe)
⚙️ AI Assistant (OpenAI)
⚙️ Email Notifications (SMTP)
⚙️ Advanced Caching (Redis)

**Time to First Scan: 5 minutes** ✅
**Time to Full Production: 1 day** ✅

---

## 🔥 Let's Go!

```bash
npm start
```

**Open:** http://localhost:3000

**Start scanning. Start securing. Start selling.** 🚀

---

**NEXUS ULTIMATE PRO v5.0**
*From Zero to Scanning in 5 Minutes*
*The Fastest Setup in Security SaaS History*

**LET'S DOMINATE!** 💎👑🔥
