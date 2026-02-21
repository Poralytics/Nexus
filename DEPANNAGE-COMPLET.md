# 🔧 GUIDE DE DÉPANNAGE COMPLET

## 🎯 PROBLÈME: "Rien ne fonctionne"

Suivez ces étapes **DANS L'ORDRE** pour identifier le problème.

---

## ✅ ÉTAPE 1: Vérifier que le serveur démarre

```bash
cd backend
npm start
```

### Résultat ATTENDU:
```
✅ NEXUS Database initialized successfully!
   📊 Tables: 39
   🔍 Indexes: 44

[INFO] Stripe initialized successfully

🛡️  NEXUS Security Scanner v2.1.0
📡  Listening on http://localhost:3000
```

### Si vous ne voyez PAS ce message:
- ❌ **Erreur**: Le serveur ne démarre pas
- **Solution**: Regardez l'erreur exacte et notez-la

### Si le serveur démarre mais s'arrête immédiatement:
- ❌ **Problème**: Crash au démarrage
- **Solution**: Vérifiez les logs d'erreur

---

## ✅ ÉTAPE 2: Tester l'API directement

**Dans un NOUVEAU terminal** (laissez le serveur tourner):

```bash
cd backend
node test-api.js
```

### Résultat ATTENDU:
```
✅ Status: 201 Created
📄 Response body:
{"success":true,"token":"eyJ...","user":{...}}

🎉 TEST RÉUSSI!
```

### Si vous voyez:
#### A) `❌ Erreur requête: connect ECONNREFUSED`
- **Cause**: Le serveur n'est PAS démarré
- **Solution**: Retour à l'étape 1

#### B) `❌ Status: 404 Not Found`
- **Cause**: La route `/api/auth/register` n'existe pas
- **Solution**: Vérifier que `routes/auth.js` est chargé dans `server.js`

#### C) `❌ Status: 500 Internal Server Error`
- **Cause**: Erreur dans le code de la route
- **Solution**: Regarder les logs du serveur (terminal où `npm start` tourne)

#### D) `⚠️  Erreur API: ...`
- **Cause**: L'API retourne une erreur spécifique
- **Solution**: Lire le message d'erreur

---

## ✅ ÉTAPE 3: Tester avec le navigateur en mode DEBUG

1. **Ouvrir**: `http://localhost:3000/register-debug.html`

2. **Remplir le formulaire** (déjà pré-rempli):
   - Name: `Test User`
   - Email: `test@example.com`
   - Password: `password123`

3. **Cliquer** "Create Account"

4. **Regarder la zone de debug** (en bas du formulaire)

### Ce que vous devriez voir:
```
[14:30:45] 🚀 Début de l'inscription
[14:30:45] 📧 Email: test@example.com
[14:30:45] 👤 Name: Test User
[14:30:45] 🔑 Password length: 11
[14:30:45] ✅ Validation OK
[14:30:45] 📦 Payload: {"email":"test@example.com",...}
[14:30:45] 🌐 URL: http://localhost:3000/api/auth/register
[14:30:45] 📡 Envoi de la requête...
[14:30:46] 📬 Réponse reçue: Status 201 Created
[14:30:46] ✅ Token reçu: eyJ...
[14:30:46] 🎉 Inscription réussie!
```

### Si vous voyez une erreur:
- Notez **EXACTEMENT** le message d'erreur
- Notez à quelle ligne ça s'arrête

---

## ✅ ÉTAPE 4: Vérifier la console du navigateur

1. **Ouvrir** la console (F12 ou Ctrl+Shift+I)
2. **Onglet "Console"**
3. **Regarder** s'il y a des erreurs en ROUGE

### Erreurs communes:

#### `net::ERR_CONNECTION_REFUSED`
- **Cause**: Le serveur n'est pas démarré
- **Solution**: Démarrer le serveur (`npm start`)

#### `Cross-Origin Request Blocked (CORS)`
- **Cause**: Problème de CORS (peu probable ici)
- **Solution**: Vérifier `server.js` ligne ~48 (CORS config)

#### `Failed to fetch`
- **Cause**: Réseau ou serveur down
- **Solution**: Vérifier que le serveur tourne

---

## ✅ ÉTAPE 5: Vérifier l'onglet Network

1. **F12** → Onglet **"Network"**
2. **Cliquer** "Create Account"
3. **Chercher** la ligne `/register` ou `/api/auth/register`
4. **Cliquer** dessus
5. **Regarder**:
   - **Headers**: Request headers, URL
   - **Payload**: Ce qui est envoyé
   - **Response**: Ce que le serveur retourne

### Que chercher:

#### Si la ligne `/register` n'apparaît PAS:
- **Cause**: La requête n'est même pas envoyée
- **Problème**: JavaScript frontend
- **Solution**: Console doit avoir une erreur

#### Si Status = 404:
- **Cause**: Route inexistante
- **Vérifier**: `server.js` ligne 110 → `app.use('/api/auth', ...)`

#### Si Status = 500:
- **Cause**: Erreur serveur
- **Vérifier**: Logs du terminal où tourne `npm start`

#### Si Status = 201 mais rien ne se passe:
- **Cause**: JavaScript frontend ne gère pas la réponse
- **Vérifier**: Console pour erreurs

---

## ✅ ÉTAPE 6: Vérifier la base de données

```bash
cd backend
ls -lh nexus-ultimate.db
```

### Si le fichier n'existe PAS:
```bash
npm run init
```

### Vérifier le contenu:
```bash
node -e "
const db = require('better-sqlite3')('./nexus-ultimate.db');
const users = db.prepare('SELECT * FROM users').all();
console.log('Users:', users.length);
users.forEach(u => console.log('  -', u.email));
"
```

---

## 🚨 CHECKLIST DE DÉPANNAGE

Cochez ce qui fonctionne:

- [ ] `npm start` démarre sans erreur
- [ ] Message "Listening on http://localhost:3000" affiché
- [ ] `node test-api.js` retourne success
- [ ] `http://localhost:3000` affiche la page d'accueil
- [ ] `http://localhost:3000/register-debug.html` affiche le formulaire
- [ ] Cliquer "Create Account" affiche des logs dans la zone debug
- [ ] Console navigateur (F12) sans erreur rouge
- [ ] Onglet Network montre la requête POST
- [ ] Status de la requête = 201 Created
- [ ] Response contient `{"success":true,"token":"..."}`

---

## 📋 RAPPORTER LE PROBLÈME

Si RIEN ne fonctionne après ces tests, donnez-moi:

1. **Ce que dit** `npm start` (copier tout le texte)
2. **Ce que dit** `node test-api.js`
3. **Ce que montre** la zone de debug sur `register-debug.html`
4. **Ce que montre** la console (F12 → Console)
5. **Ce que montre** Network (F12 → Network → clic sur /register)

Avec ces infos, je pourrai identifier le problème EXACT.

---

## 🎯 SOLUTIONS RAPIDES

### "Le serveur ne démarre pas"
```bash
# Supprimer node_modules et réinstaller
rm -rf node_modules package-lock.json
npm install
```

### "Email already registered"
```bash
# Supprimer l'utilisateur de test
cd backend
node -e "
const db = require('better-sqlite3')('./nexus-ultimate.db');
db.prepare('DELETE FROM users WHERE email = ?').run('test@example.com');
console.log('Utilisateur supprimé');
"
```

### "Database locked"
```bash
# Fermer tous les processus qui utilisent la DB
# Windows
taskkill /F /IM node.exe

# Linux/Mac
pkill node
```

---

**Suivez ces étapes et notez EXACTEMENT où ça bloque!**
