# ⚽ Cloud Quiz Foot - Groupe 11

Cloud Quiz Foot est une application web cloud-native permettant de jouer à un quiz de football généré dynamiquement.  
Le projet utilise Azure (App Service, Functions, Table Storage), de l'Infrastructure as Code (Bicep) et un déploiement manuel via Azure CLI.

---

## 📌 1. Objectif du projet  
Cloud Quiz Foot a pour objectif de démontrer la mise en place d'une architecture cloud moderne et scalable à travers une application simple et ludique.  
Le projet illustre :

- Le déploiement d'une application web sur Azure  
- La création d'un backend serverless via Azure Functions  
- L'utilisation d'un stockage cloud (Table Storage)  
- La gestion complète de l'infrastructure via IaC (Bicep)  
- Une séparation claire frontend / backend / infra

---

## 📂 2. Architecture du projet

### 🧱 Structure générale
```
cloud-quiz-foot/
├── frontend/           → Interface web (HTML/CSS/JS)
├── Backend/            → Azure Functions (API serverless Python)
│   ├── generatequiz/   → Génère un quiz aléatoire
│   ├── nextquestion/   → Récupère la question suivante
│   ├── submitresult/   → Enregistre un score
│   ├── getleaderboard/ → Récupère le classement
│   └── scripts/        → Script d'import des questions
├── infra/              → Infrastructure as Code (Bicep)
└── README.md
```

### ☁️ Composants Azure utilisés

| Service Azure | Rôle |
|---------------|------|
| **App Service** | Hébergement du frontend |
| **Azure Functions** | API serverless (Python) |
| **Storage Account** | Stockage des questions et scores (Table Storage) |
| **App Service Plan (F1)** | Plan gratuit pour le frontend |
| **Function Plan (Y1)** | Plan Consumption pour les Functions |

---

## 🚀 3. Déploiement complet from scratch

### Prérequis

1. **Compte Azure** avec un abonnement actif
2. **Azure CLI** installé : https://aka.ms/installazurecliwindows
3. **Azure Functions Core Tools** :
   ```bash
   npm install -g azure-functions-core-tools@4 --unsafe-perm true
   ```
4. **Python 3.11** installé
5. **Git** installé

---

### Étape 1 : Cloner le projet

```bash
git clone https://github.com/Titouanglangetas/cloud-quiz-foot.git
cd cloud-quiz-foot
```

---

### Étape 2 : Se connecter à Azure

```bash
az login
```

Une fenêtre de navigateur s'ouvrira pour vous authentifier.

---

### Étape 3 : Créer le Resource Group

```bash
az group create --name CloudQuizFootRG --location uksouth
```

> ⚠️ Choisissez une région autorisée par votre abonnement ou des erreurs apparaîtront à l'étape suivante. Régions courantes : `uksouth`, `westeurope`, `francecentral`

---

### Étape 4 : Déployer l'infrastructure avec Bicep

```bash
az deployment group create --resource-group CloudQuizFootRG --template-file infra/main.bicep
```

Cette commande crée automatiquement :
- ✅ Storage Account avec Table Storage
- ✅ Tables `questions` et `scores`
- ✅ App Service Plan (frontend)
- ✅ App Service (frontend)
- ✅ Function Plan (backend)
- ✅ Function App (backend Linux Python)

---

### Étape 5 : Récupérer la connection string du Storage

```bash
az storage account show-connection-string --name cloudquizfootprojstor --resource-group CloudQuizFootRG --query connectionString -o tsv
```

Copiez cette valeur et mettez-la dans `Backend/local.settings.json` :

```json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "AzureWebJobsStorage": "<VOTRE_CONNECTION_STRING>",
    "TABLE_STORAGE_CONNECTION_STRING": "<VOTRE_CONNECTION_STRING>"
  }
}
```

---

### Étape 6 : Importer les questions dans Table Storage

```bash
cd Backend/scripts
pip install azure-data-tables
python create_questions.py
```

Vous devriez voir :
```
🔎 Chargement de : ...\local.settings.json
🔗 Connexion au storage OK
📄 150 questions chargées.
📥 Import en cours...
✅ Import terminé !
```

---

### Étape 7 : Déployer le Backend (Azure Functions)

```bash
cd ..
func azure functionapp publish cloudquizfootproj-functions --build local
```

Après déploiement, vous verrez les URLs :
```
Functions in cloudquizfootproj-functions:
    generatequiz - https://cloudquizfoot2-functions.azurewebsites.net/api/generatequiz
    getleaderboard - https://cloudquizfoot2-functions.azurewebsites.net/api/getleaderboard
    nextquestion - https://cloudquizfoot2-functions.azurewebsites.net/api/nextquestion
    submitresult - https://cloudquizfoot2-functions.azurewebsites.net/api/submitresult
```

---

### Étape 8 : Configurer l'URL de l'API dans le Frontend

Modifiez `frontend/script.js` (ligne 2) :

```javascript
const API_BASE_URL = "https://cloudquizfootproj-functions.azurewebsites.net/api";
```

Modifiez aussi `frontend/leaderboard.js` (ligne 1) :

```javascript
const API = "https://cloudquizfootproj-functions.azurewebsites.net/api";
```

---

### Étape 9 : Déployer le Frontend

**Windows PowerShell :**
```powershell
Compress-Archive -Path frontend\* -DestinationPath frontend.zip -Force
az webapp deploy --resource-group CloudQuizFootRG --name cloudquizfootproj-frontend --src-path frontend.zip --type zip
```

**Bash/Linux/Mac :**
```bash
zip -r frontend.zip frontend/*
az webapp deploy --resource-group CloudQuizFootRG --name cloudquizfootproj-frontend --src-path frontend.zip --type zip
```

---

### Étape 10 : Configurer CORS

Autorisez le frontend à appeler le backend :

```bash
az functionapp cors add --name cloudquizfootproj-functions --resource-group CloudQuizFootRG --allowed-origins "https://cloudquizfootproj-frontend.azurewebsites.net"
```

---

## ✅ 4. Tester l'application

### URLs de production

| Composant | URL |
|-----------|-----|
| **Frontend** | https://cloudquizfootproj-frontend.azurewebsites.net |
| **API generatequiz** | https://cloudquizfootproj-functions.azurewebsites.net/api/generatequiz |
| **API nextquestion** | https://cloudquizfootproj-functions.azurewebsites.net/api/nextquestion |
| **API getleaderboard** | https://cloudquizfootproj-functions.azurewebsites.net/api/getleaderboard |
| **API submitresult** | https://cloudquizfootproj-functions.azurewebsites.net/api/submitresult |

---

## 🧪 5. Développement local

### Lancer le backend localement

```bash
cd Backend
python -m venv .venv
.venv\Scripts\Activate.ps1   # Windows
# ou: source .venv/bin/activate   # Linux/Mac
pip install -r requirements.txt
func start
```

Le backend sera disponible sur `http://localhost:7071/api`

### Tester le frontend localement

Ouvrez `frontend/index.html` dans un navigateur, ou utilisez Live Server dans VS Code.

> ⚠️ Pour le dev local, modifiez `API_BASE_URL` dans `script.js` :
> ```javascript
> const API_BASE_URL = "http://localhost:7071/api";
> ```

---

## ⚙️ 6. Backend : Azure Functions (Python)

| Function | Méthode | Description |
|----------|---------|-------------|
| `generatequiz` | GET | Renvoie 5 questions aléatoires |
| `nextquestion` | GET | Renvoie une question selon la difficulté |
| `submitresult` | POST | Enregistre un score dans la table `scores` |
| `getleaderboard` | GET | Renvoie le top 10 des joueurs |

### Paramètres de nextquestion

```
GET /api/nextquestion?difficulty=3&used=1,2,3
```

- `difficulty` : niveau de difficulté (1-10)
- `used` : IDs des questions déjà posées (évite les doublons)

---

## 🧰 7. Infrastructure as Code (Bicep)

Le fichier `infra/main.bicep` crée toutes les ressources Azure :

```bicep
// Paramètres
param projectName string = 'cloudquizfootproj'
param location string = 'uksouth'

// Ressources créées :
// - Storage Account + Table Storage (questions, scores)
// - App Service Plan (F1 gratuit)
// - App Service (frontend)
// - Function Plan (Y1 Consumption)
// - Function App (Python 3.11 Linux)
```

Pour redéployer l'infrastructure :
```bash
az deployment group create --resource-group CloudQuizFootRG --template-file infra/main.bicep
```

---

## 🗑️ 8. Nettoyage des ressources

Pour supprimer toutes les ressources Azure :

```bash
az group delete --name CloudQuizFootRG --yes --no-wait
```

---

## 🗃️ 9. Format des données

### Questions (Table Storage)

```json
{
  "PartitionKey": "Q",
  "RowKey": "1",
  "difficulty": 1,
  "question": "Quel pays a remporté la Coupe du Monde 2018 ?",
  "choice1": "France",
  "choice2": "Croatie",
  "choice3": "Belgique",
  "answer": "France"
}
```

### Scores (Table Storage)

```json
{
  "PartitionKey": "score",
  "RowKey": "uuid",
  "name": "Joueur1",
  "score": 85,
  "mode": "qifoot",
  "timestamp": "2025-12-12T17:00:00Z"
}
```

---

## 👥 10. Équipe

- Titouan Glangetas
- Arthur Fatus
- Quentin Petiteville
