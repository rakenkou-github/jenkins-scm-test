# 🚀 Guide de Démarrage - Jenkins avec Docker sur Windows

## 📋 Architecture

```
Windows (Docker Desktop)
    │
    ├─ Jenkins Container (avec Docker CLI)
    │   ├─ Accès au Docker Engine via /var/run/docker.sock
    │   └─ Construit les images : pyspark-agent, python-agent
    │
    ├─ Agents Docker (créés dynamiquement par Jenkins)
    │   ├─ pyspark-agent (Apache Spark + Python)
    │   └─ python-agent (Python léger)
    │
    └─ Docker Engine (gère tous les conteneurs)
```

## 🔧 Configuration : Docker-out-of-Docker (DooD)

### Comment ça fonctionne ?

1. **Jenkins** s'exécute dans un conteneur
2. Le **socket Docker** (`/var/run/docker.sock`) est monté depuis l'hôte Windows
3. **Docker CLI** est installé dans le conteneur Jenkins
4. Jenkins utilise le Docker CLI pour communiquer avec le **Docker Engine de l'hôte**
5. Les images et conteneurs sont créés **directement sur Docker Desktop**

### Avantages

✅ Pas de Docker-in-Docker complexe  
✅ Meilleure performance  
✅ Les images construites sont disponibles sur l'hôte  
✅ Partage du réseau Docker entre Jenkins et les agents  

## 🚀 Démarrage Rapide

### Étape 1 : Construire et démarrer Jenkins

```powershell
# Naviguer vers le dossier Jenkins
cd C:\Formation\Learning-2026\Jenkins

# Construire l'image Jenkins personnalisée avec Docker CLI
docker-compose build

# Démarrer Jenkins
docker-compose up -d

# Vérifier que Jenkins est démarré
docker-compose ps
```

### Étape 2 : Récupérer le mot de passe initial

```powershell
# Attendre 30-60 secondes que Jenkins démarre complètement
Start-Sleep -Seconds 60

# Afficher le mot de passe initial
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### Étape 3 : Accéder à Jenkins

Ouvrez votre navigateur : `http://localhost:8080`

### Étape 4 : Vérifier que Docker fonctionne dans Jenkins

```powershell
# Tester que Docker CLI est accessible depuis Jenkins
docker exec jenkins docker --version
docker exec jenkins docker ps
docker exec jenkins docker images
```

Vous devriez voir les images et conteneurs de votre Docker Desktop !

## 🔍 Vérifications Importantes

### 1. Docker Desktop doit être en cours d'exécution

```powershell
# Vérifier que Docker Desktop fonctionne
docker ps
```

### 2. Le socket Docker doit être accessible

Dans Docker Desktop :
- **Settings > General**
- Cocher **"Expose daemon on tcp://localhost:2375 without TLS"** (si besoin)

### 3. Permissions sur le socket Docker

Le conteneur Jenkins s'exécute en `root` pour avoir accès au socket.

## 📝 Configuration du Pipeline Jenkins

### 1. Créer un nouveau Pipeline

1. Cliquez sur **"Nouveau Item"**
2. Nom : `pyspark-csv-pipeline`
3. Type : **Pipeline**
4. Cliquez sur **OK**

### 2. Configuration Pipeline depuis Git

Dans **Pipeline** :

**Option A : Pipeline script from SCM (Recommandé)**

```
Definition: Pipeline script from SCM
SCM: Git
Repository URL: https://github.com/VOTRE_USER/VOTRE_REPO.git
Branch: */main
Script Path: pyspark-csv-project/Jenkinsfile
```

**Option B : Pipeline script direct (pour tester)**

Copiez le contenu du `Jenkinsfile` directement dans la zone de texte.

### 3. Première exécution

1. Cliquez sur **"Build Now"**
2. Le pipeline va :
   - Construire les images Docker (`pyspark-agent`, `python-agent`)
   - Exécuter les traitements en parallèle
   - Générer des rapports

## 🐛 Résolution des Problèmes

### Problème 1 : "Cannot connect to Docker daemon"

**Cause** : Jenkins ne peut pas accéder au socket Docker

**Solution** :

```powershell
# Vérifier que le socket est monté
docker inspect jenkins | Select-String "docker.sock"

# Redémarrer Jenkins
docker-compose restart jenkins

# Vérifier à nouveau
docker exec jenkins docker ps
```

### Problème 2 : "sh: docker: command not found"

**Cause** : Docker CLI n'est pas installé dans Jenkins

**Solution** : Reconstruire l'image Jenkins

```powershell
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

### Problème 3 : "Permission denied" sur docker.sock

**Cause** : Permissions insuffisantes

**Solution** :

```powershell
# Le conteneur doit s'exécuter en root
# Vérifier dans docker-compose.yml :
# user: root

# Redémarrer
docker-compose restart jenkins
```

### Problème 4 : Les images ne se construisent pas

**Cause** : Chemin incorrect ou fichiers manquants

**Solution** : Vérifier le montage du projet

```powershell
# Vérifier que le projet est accessible depuis Jenkins
docker exec jenkins ls -la /workspace/pyspark-csv-project

# Si vide, vérifier le volume dans docker-compose.yml
```

### Problème 5 : "Dockerfile not found"

**Cause** : Le Jenkinsfile cherche dans le mauvais répertoire

**Solution** : Dans le Jenkinsfile, le `dir('pyspark-csv-project')` assume que le projet est dans le workspace Jenkins.

Si vous utilisez Git, le code est automatiquement cloné dans le workspace.

Si vous testez localement sans Git, modifiez le Jenkinsfile :

```groovy
stage('Build Docker Images') {
    agent any
    steps {
        echo "🐳 Construction des images Docker pour les agents..."
        script {
            // Utiliser le chemin monté
            dir('/workspace/pyspark-csv-project') {
                sh 'docker build -t pyspark-agent:latest -f Dockerfile.pyspark .'
                sh 'docker build -t python-agent:latest -f Dockerfile.python .'
            }
        }
        echo "✅ Images Docker construites avec succès"
    }
}
```

## 📊 Vérifier les Images et Conteneurs

```powershell
# Images disponibles
docker images | Select-String "pyspark|python-agent|jenkins"

# Conteneurs actifs
docker ps

# Logs Jenkins
docker logs -f jenkins

# Logs d'un agent spécifique
docker ps  # Noter l'ID
docker logs <CONTAINER_ID>
```

## 🎯 Test Complet

### Script de test automatique

```powershell
# Script de test complet
Write-Host "🔍 Vérification de l'environnement..." -ForegroundColor Cyan

# 1. Docker Desktop
Write-Host "`n1️⃣ Docker Desktop:" -ForegroundColor Yellow
docker --version

# 2. Jenkins en cours d'exécution
Write-Host "`n2️⃣ Jenkins:" -ForegroundColor Yellow
docker ps --filter "name=jenkins" --format "table {{.Names}}\t{{.Status}}"

# 3. Docker CLI dans Jenkins
Write-Host "`n3️⃣ Docker CLI dans Jenkins:" -ForegroundColor Yellow
docker exec jenkins docker --version

# 4. Accès au socket Docker
Write-Host "`n4️⃣ Accès au Docker Engine:" -ForegroundColor Yellow
docker exec jenkins docker ps

# 5. Projet monté
Write-Host "`n5️⃣ Projet monté:" -ForegroundColor Yellow
docker exec jenkins ls -la /workspace/pyspark-csv-project/

Write-Host "`n✅ Tous les tests sont OK!" -ForegroundColor Green
```

## 🔐 Sécurité

⚠️ **Attention** : Exécuter Jenkins en `root` avec accès au socket Docker donne un contrôle total sur Docker.

**En production** :
- Utiliser des credentials Jenkins pour les secrets
- Limiter les permissions
- Utiliser un réseau Docker isolé
- Scanner les images pour les vulnérabilités

## 📚 Ressources

- [Docker-out-of-Docker (DooD)](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/)
- [Jenkins Docker Plugin](https://plugins.jenkins.io/docker-plugin/)
- [PySpark Documentation](https://spark.apache.org/docs/latest/api/python/)

## 🎉 Prochaines Étapes

1. ✅ Configurer Git et pousser le projet
2. ✅ Configurer les webhooks Git
3. ✅ Exécuter le premier build
4. ✅ Consulter les rapports générés
5. ✅ Améliorer le pipeline (tests, notifications, etc.)

---

**Bon apprentissage ! 🚀**
