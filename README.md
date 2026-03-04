# TP3 Pipeline DevSecOps avec GitHub Actions

![Security](https://github.com/Ayoub-HM/TP3_Pipeline-DevSecOps-avec-GitHub-Actions/actions/workflows/security.yml/badge.svg)

## 1) Presentation du projet

Ce projet est un mini-lab DevSecOps.
Il combine :

- une API Node.js/Express securisee (`src/server.js`)
- un frontend statique de demonstration (`frontend/build/index.html`)
- une pipeline GitHub Actions qui automatise build, scans securite, rapport, et deploiement frontend sur GitHub Pages.

Objectif principal : montrer comment integrer la securite dans le cycle CI/CD (shift-left security) au lieu de la traiter seulement a la fin.

## 2) Ce que fait l'application

### Backend (API)

L'API expose :

- `GET /health` : endpoint de sante
- `POST /api/login` : endpoint de connexion qui retourne un JWT si les identifiants sont valides

Mesures de securite implementees :

- `helmet` pour les headers HTTP de securite
- `express-rate-limit` pour limiter les tentatives de login
- `express-validator` pour valider les entrees utilisateur
- secret JWT obligatoire via variable d'environnement (`JWT_SECRET`, min 32 caracteres)
- mode debug desactive en production

### Frontend

Le frontend actuel est un placeholder statique.
Il est deploie sur GitHub Pages via la pipeline.

## 3) Stack et outils utilises

### Application

- Node.js + Express
- jsonwebtoken
- helmet
- express-rate-limit
- express-validator
- dotenv

### Conteneurisation

- Docker (`Dockerfile`)
- Image de base `node:22-alpine`
- Execution en utilisateur non-root
- Healthcheck Docker sur `/health`

### DevSecOps (CI/CD)

Workflow : `.github/workflows/security.yml`

Outils utilises dans la pipeline :

- **Semgrep (SAST)** : analyse statique du code source (regles security-audit, secrets, OWASP top 10)
- **npm audit (SCA)** : detection des vulnerabilites dans les dependances npm
- **Gitleaks** : detection de secrets exposes dans l'historique Git
- **Trivy** : scan de vulnerabilites de l'image Docker
- **OWASP ZAP Baseline (DAST)** : scan dynamique de l'application en execution
- **GitHub Pages Actions** : publication automatique du frontend

## 4) Structure du repo

```text
.
|-- .github/workflows/security.yml
|-- src/
|   |-- server.js
|   `-- package.json
|-- frontend/build/index.html
|-- Dockerfile
|-- .env.example
`-- README.md
```

## 5) Variables d'environnement

Exemple dans `.env.example` :

```env
JWT_SECRET=generate-a-strong-random-secret-min-32-chars
ADMIN_USER=admin
ADMIN_PASS=strong-password-here
NODE_ENV=production
```

Variables obligatoires pour l'API :

- `JWT_SECRET` (>= 32 caracteres)
- `ADMIN_USER`
- `ADMIN_PASS`

## 6) Lancer en local (sans Docker)

```bash
cd src
npm install
```

Creer un fichier `.env` a la racine du projet (ou exporter les variables dans votre shell), puis lancer :

```bash
node server.js
```

Test rapide :

```bash
curl http://localhost:3000/health
```

## 7) Lancer avec Docker

Build :

```bash
docker build -t vuln-app:local .
```

Run :

```bash
docker run --rm -p 3000:3000 \
  -e JWT_SECRET="your-very-strong-secret-at-least-32-chars" \
  -e ADMIN_USER="admin" \
  -e ADMIN_PASS="strong-password" \
  -e NODE_ENV="production" \
  vuln-app:local
```

## 8) Pipeline GitHub Actions (resume des jobs)

Declenchement :

- `push`
- `pull_request`

Jobs principaux :

1. `build` : build de l'image Docker + upload artifact (`image.tar`)
2. `sast` : scan statique Semgrep
3. `sca` : audit npm des dependances
4. `secrets` : scan Gitleaks
5. `container-scan` : scan Trivy de l'image Docker
6. `dast` : lancement du conteneur + scan OWASP ZAP
7. `deploy` : deploiement du frontend sur GitHub Pages (branche `main` uniquement)
8. `report` : generation d'un rapport JSON et resume final

## 9) Deploiement GitHub Pages

Le job `deploy` publie `./frontend/build` sur GitHub Pages.

Points importants :

- le job tourne seulement sur `main`
- il faut autoriser les permissions Actions en `Read and write`
- `Settings > Pages > Source` doit etre `GitHub Actions`

## 10) Artifacts generes par la pipeline

- `docker-image` : image Docker exportee (`image.tar`)
- `npm-audit` : resultat `audit.json`
- `zap-scan` : artefacts OWASP ZAP
- `security-report` : rapport JSON final

## 11) Limites actuelles

- frontend minimal (placeholder)
- pas de base de donnees
- pas de suite de tests unitaires/integration
- scan Trivy configure sur severite `CRITICAL` uniquement

## 12) Pistes d'amelioration

- ajouter des tests automatiques (unitaires + integration + e2e)
- ajouter un SBOM (ex: CycloneDX) et signature d'image (ex: cosign)
- renforcer les politiques de merge (branch protection + required checks)
- remplacer le frontend placeholder par un vrai client web

---

Ce README sert de reference rapide pour comprendre le projet, la securisation de l'API, et le role de chaque etape de la pipeline DevSecOps.
