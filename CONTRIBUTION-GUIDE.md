# 🤝 NEXUS ULTIMATE PRO - Contribution Guide

## Comment Contribuer au Projet

---

## 🎯 FAÇONS DE CONTRIBUER

### 1. Code
- Ajouter nouveaux scanners
- Améliorer scanners existants
- Optimisations performance
- Corrections bugs
- Nouvelles fonctionnalités

### 2. Documentation
- Améliorer guides
- Traduire en d'autres langues
- Ajouter exemples
- Tutoriels vidéo
- Cas d'usage

### 3. Tests
- Tests unitaires
- Tests intégration
- Tests E2E
- Tests performance
- Rapports bugs

### 4. Design
- Améliorer UI/UX
- Créer mockups
- Icônes et assets
- Thèmes
- Accessibilité

### 5. Community
- Répondre aux questions
- Modération Discord/Slack
- Organisation événements
- Mentoring nouveaux contributeurs

---

## 📋 PROCESSUS DE CONTRIBUTION

### Étape 1: Fork & Clone

```bash
# Fork sur GitHub
# Puis clone ton fork
git clone https://github.com/TON-USERNAME/nexus-ultimate-pro
cd nexus-ultimate-pro

# Ajouter upstream
git remote add upstream https://github.com/nexus-security/nexus-ultimate-pro
```

### Étape 2: Créer Branch

```bash
# Toujours créer une branche pour tes changes
git checkout -b feature/nouveau-scanner-ldap
# ou
git checkout -b fix/correction-sql-scanner
# ou
git checkout -b docs/guide-francais
```

**Convention nommage branches:**
- `feature/` - Nouvelles fonctionnalités
- `fix/` - Corrections bugs
- `docs/` - Documentation
- `test/` - Tests
- `refactor/` - Refactoring
- `perf/` - Optimisations

### Étape 3: Développer

```bash
# Installer dépendances
cd backend
npm install

# Créer ta feature
# Exemple: Nouveau scanner
touch backend/scanners/ldap-injection-scanner.js

# Tester localement
npm test
npm start

# Tester manuellement
```

### Étape 4: Commit

```bash
# Convention commits (Conventional Commits)
git add .
git commit -m "feat(scanners): add LDAP injection scanner"
# ou
git commit -m "fix(sql-scanner): correct time-based detection"
# ou
git commit -m "docs(readme): add French translation"
```

**Convention messages commit:**
```
type(scope): description courte

[body optionnel]

[footer optionnel]
```

**Types:**
- `feat` - Nouvelle fonctionnalité
- `fix` - Correction bug
- `docs` - Documentation
- `style` - Formatage code
- `refactor` - Refactoring
- `test` - Tests
- `chore` - Maintenance

### Étape 5: Push & Pull Request

```bash
# Push ta branche
git push origin feature/nouveau-scanner-ldap

# Sur GitHub, créer Pull Request
# Titre: "Add LDAP injection scanner"
# Description:
## Changes
- Nouveau scanner LDAP injection
- Détecte 3 types d'attaques
- 150 lignes de code
- 95% coverage

## Testing
- ✓ Tests unitaires
- ✓ Tests intégration
- ✓ Tests manuels sur ldap.forumsys.com

## Screenshots
[screenshots si UI]
```

### Étape 6: Code Review

```markdown
# Répondre aux commentaires
# Faire les ajustements
git add .
git commit -m "fix: address review comments"
git push origin feature/nouveau-scanner-ldap

# Une fois approuvé: Merge!
```

---

## 💻 STANDARDS CODE

### JavaScript Style Guide

```javascript
// ✅ BON
class LDAPScanner {
  constructor(domain) {
    this.domain = domain;
    this.findings = [];
  }

  async scan() {
    await this.testAuthentication();
    await this.testInjection();
    return this.findings;
  }

  async testAuthentication() {
    try {
      const response = await axios.get(this.domain.url);
      if (this.detectLDAPError(response.data)) {
        this.findings.push({
          severity: 'high',
          category: 'ldap_injection',
          title: 'LDAP Injection Vulnerability',
          // ...
        });
      }
    } catch (error) {
      console.error('LDAP test error:', error);
    }
  }
}

// ❌ MAUVAIS
class ldap_scanner {  // PascalCase for classes
  constructor(d) {  // Descriptive names
    this.d = d;  // Single letter variables
    this.f = [];
  }

  async scan() {  // No error handling
    let r = await axios.get(this.d.url);
    if (r.data.includes('ldap')) {
      this.f.push({title: 'LDAP vuln'});  // Incomplete
    }
  }
}
```

### Code Quality Checklist

```markdown
- [ ] Code suit ESLint rules
- [ ] Pas de console.log (sauf debug)
- [ ] Error handling partout
- [ ] Variables descriptives
- [ ] Functions < 50 lignes
- [ ] Comments sur logique complexe
- [ ] Pas de code dupliqué
- [ ] Tests unitaires ajoutés
- [ ] Documentation mise à jour
```

### ESLint Configuration

```javascript
// .eslintrc.js
module.exports = {
  env: {
    node: true,
    es2021: true
  },
  extends: 'eslint:recommended',
  rules: {
    'indent': ['error', 2],
    'quotes': ['error', 'single'],
    'semi': ['error', 'always'],
    'no-unused-vars': 'error',
    'no-console': 'warn'
  }
};
```

---

## 🧪 TESTS REQUIS

### Pour Nouveau Scanner

```javascript
// tests/scanners/ldap-injection.test.js
describe('LDAP Injection Scanner', () => {
  test('should detect LDAP errors', () => {
    // Test implementation
  });

  test('should find injection points', () => {
    // Test implementation
  });

  test('should calculate CVSS correctly', () => {
    // Test implementation
  });
});
```

**Minimum Coverage: 80%**

---

## 📝 DOCUMENTATION REQUISE

### Pour Nouveau Scanner

```markdown
# docs/scanners/ldap-injection.md

## LDAP Injection Scanner

### Description
Détecte les vulnérabilités d'injection LDAP...

### Tests Effectués
1. Authentication bypass
2. Filter injection
3. DN injection

### Payloads
- `*)(uid=*))(|(uid=*`
- `admin*`
- `*)(objectClass=*`

### Détection
- Erreurs LDAP dans réponse
- Changements comportement
- Timing attacks

### False Positives
- Applications utilisant LDAP légitimement
- Erreurs génériques

### Références
- OWASP Testing Guide
- CWE-90
```

---

## 🎨 DESIGN GUIDELINES

### UI Components

```javascript
// Consistent styling
<button className="btn btn-primary">Scanner</button>
<button className="btn btn-secondary">Annuler</button>
<button className="btn btn-danger">Supprimer</button>

// Color palette
--color-primary: #3b82f6;
--color-danger: #ef4444;
--color-success: #10b981;
--color-warning: #f59e0b;

// Spacing
margin: 1rem;  // 16px
padding: 0.5rem;  // 8px
gap: 1.5rem;  // 24px
```

---

## 🚀 PROCESSUS RELEASE

### Version Numbering (SemVer)

```
MAJOR.MINOR.PATCH

2.1.0
│ │ │
│ │ └─ Patch: Corrections bugs
│ └─── Minor: Nouvelles features (backward compatible)
└───── Major: Breaking changes
```

### Checklist Release

```markdown
## Pre-Release
- [ ] Tous tests passent
- [ ] Pas de console.log
- [ ] Documentation à jour
- [ ] CHANGELOG.md mis à jour
- [ ] Version bumped (package.json)

## Release
- [ ] Tag créé (git tag v2.1.0)
- [ ] GitHub Release créée
- [ ] Docker images pushed
- [ ] NPM package publié (si applicable)

## Post-Release
- [ ] Annonce sur Discord/Twitter
- [ ] Documentation déployée
- [ ] Migration guide (si breaking changes)
```

---

## 🌍 TRADUCTION

### Ajouter Nouvelle Langue

```bash
# 1. Créer fichier traduction
cp i18n/en.json i18n/fr.json

# 2. Traduire strings
{
  "dashboard.title": "Tableau de Bord",
  "scan.start": "Démarrer le Scan",
  "vulnerabilities.critical": "Critique"
}

# 3. Importer dans app
import fr from './i18n/fr.json';
```

---

## 🏆 RECONNAISSANCE

### Hall of Fame

Contributeurs avec le plus d'impact:

1. **Top Contributor** - 50+ commits
2. **Bug Hunter** - 20+ bugs reportés
3. **Documentation Hero** - 10+ docs améliorées
4. **Test Champion** - Coverage +10%
5. **Community Leader** - 100+ questions répondues

### Badges

- 🥇 Core Contributor
- 🥈 Active Contributor
- 🥉 Contributor
- 🐛 Bug Hunter
- 📚 Documentarian
- 🧪 Test Engineer
- 🎨 Designer
- 🌍 Translator

---

## 💬 COMMUNICATION

### Channels

**GitHub Issues** - Bugs & features
**Discord** - Chat temps réel
**Forum** - Discussions longues
**Twitter** - Annonces
**Email** - Contact direct

### Response Times

- Critical bugs: 24h
- Regular bugs: 7 jours
- Feature requests: 14 jours
- Questions: 48h

---

## 📜 CODE OF CONDUCT

### Nos Valeurs

1. **Respect** - Traiter chacun avec dignité
2. **Inclusivité** - Tout le monde est bienvenu
3. **Collaboration** - Travailler ensemble
4. **Excellence** - Viser la qualité
5. **Transparence** - Communication ouverte

### Comportements Inacceptables

❌ Harcèlement
❌ Discrimination
❌ Trolling
❌ Spam
❌ Divulgation informations privées

### Conséquences

1. Avertissement
2. Suspension temporaire
3. Ban permanent

---

## 🎓 RESSOURCES POUR CONTRIBUTEURS

### Learning

- **JavaScript**: MDN Web Docs
- **Node.js**: nodejs.org/docs
- **Security**: OWASP Top 10
- **Testing**: Jest documentation

### Tools

- **IDE**: VS Code (recommended)
- **Git GUI**: GitKraken / SourceTree
- **API Testing**: Postman / Insomnia
- **DB Client**: DBeaver

---

## 📊 MÉTRIQUES

### Contribution Goals 2024

- [ ] 100+ contributeurs
- [ ] 1000+ stars GitHub
- [ ] 50+ issues fermées
- [ ] 95%+ coverage tests
- [ ] 10+ langues supportées

---

## ✉️ CONTACT

**Maintainers**
- @maintainer1 - Core features
- @maintainer2 - Security
- @maintainer3 - Documentation

**Email**: contributors@nexus-security.com
**Discord**: discord.gg/nexus-contrib

---

**Merci de contribuer à NEXUS ULTIMATE PRO ! 🎉**

Ensemble, nous créons le meilleur scanner de sécurité open source.
