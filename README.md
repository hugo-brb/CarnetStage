🌍 English version available [here](README.en.md)

# 📘 CarnetStage

## 🚀 Prérequis

Avant de commencer, assurez-vous d’avoir installé les outils suivants sur votre machine :

- 🐳 [**Docker**](https://www.docker.com/get-started)  
- 🐙 [**Docker Compose**](https://docs.docker.com/compose/install/) (généralement inclus avec Docker)  
- 🔧 [**Git**](https://git-scm.com/)

---

## 🌐 Accès au serveur distant

Le projet est déployé via `docker-compose` sur un VPS. Le back-office est accessible à l’adresse suivante :

🔗 **https://156d-51-83-75-226.ngrok-free.app**

### ➕ Exécution en local (sans Docker)

Si vous souhaitez exécuter le back-office localement sans utiliser Docker (par exemple pour lancer des tests), modifiez le fichier `.env` comme suit :

1. **Commentez** la ligne par défaut :
```env
DATABASE_URL="pgsql://app-stages:app-stages@db:5432/stage_db"
```

2. **Décommentez** la ligne suivante :
```env
DATABASE_URL="pgsql://app-stages:app-stages@51.83.75.226:5432/stage_db?serverVersion=14&charset=utf8"
```

---

## 📱 Connexion de l'application mobile

Par défaut, l’application mobile est configurée pour se connecter automatiquement à l’URL du VPS :  
🔗 **https://156d-51-83-75-226.ngrok-free.app**

> ⚠️ **Recommandation :**  
Si vous exécutez le back-office en local via Docker, utilisez l’URL suivante dans l’application mobile :  
`http://10.0.2.2:8000`

Cela garantit que les actions effectuées dans l'app mobile affectent la base de données locale (et non celle du VPS).

🛠️ Cette configuration se modifie dans le fichier Android suivant :  
`api/ApiClient.java`, à la **ligne 34**.

---

## 🧪 Lancer le projet en local

### 1. 📂 Cloner le dépôt

```bash
git clone --recursive https://github.com/hugo-brb/CarnetStage.git
cd CarnetStage
```

### 2. ⚙️ Vérifier le fichier `.env`

Dans le dossier `carnet-stage-server-24-25/`, assurez-vous que le fichier `.env` est présent et correctement configuré.

Sinon, créez-le avec le contenu suivant :

```env
APP_ENV=dev
APP_DEBUG=true
APP_SECRET=changeme
######################################################################################################
# A METTRE POUR L'ENVIRONNEMENT DE "PROD"
DATABASE_URL="pgsql://app-stages:app-stages@db:5432/stage_db"
######################################################################################################
# URL POUR EXECUTER LES TESTS EN LOCAL
# DATABASE_URL="pgsql://app-stages:app-stages@51.83.75.226:5432/stage_db?serverVersion=14&charset=utf8"
######################################################################################################
TRUSTED_PROXIES=127.0.0.1,REMOTE_ADDR
TRUSTED_HOSTS=*
MAILER_DSN=null://null
CORS_ALLOW_ORIGIN=*
MESSENGER_TRANSPORT_DSN=doctrine://default
###> lexik/jwt-authentication-bundle ###
JWT_SECRET_KEY=%kernel.project_dir%/config/jwt/private.pem
JWT_PUBLIC_KEY=%kernel.project_dir%/config/jwt/public.pem
JWT_PASSPHRASE=prankex2025_soupex
###< lexik/jwt-authentication-bundle ###
```

### 3. 🏗️ Démarrer les conteneurs Docker

Lancez Docker Desktop puis, dans le dossier `carnet-stage-server-24-25/`, exécutez :

```bash
docker-compose up --build -d
```

Cette commande :

- construit les images Docker nécessaires  
- démarre les services Symfony + PostgreSQL en arrière-plan

### 4. 📦 Installer les dépendances PHP

Une fois les conteneurs lancés :

```bash
docker-compose exec backoffice composer install --no-interaction --optimize-autoloader
```

### 5. 🧱 Appliquer les migrations de la base de données

```bash
docker-compose exec backoffice php bin/console doctrine:migrations:migrate --no-interaction
```

### 5.b 🔄 Remplir la base de données (si nécessaire)

Vérifiez si des données sont présentes :

```bash
docker-compose exec db psql -U app-stages -d stage_db -c "SELECT * FROM compte_etudiant;"
```

Si la table est vide, exécutez :

```bash
docker-compose exec db psql -U app-stages -d stage_db -c "SET session_replication_role = 'replica';"
docker-compose exec -T db psql -U app-stages -d stage_db < dump_INSERT.sql
docker-compose exec -T db psql -U app-stages -d stage_db < dump_INSERT.sql
docker-compose exec db psql -U app-stages -d stage_db -c "SET session_replication_role = 'origin';"
```

### 6. 🔐 Générer les clés JWT

Pour permettre l’authentification avec l’application mobile :

```bash
docker exec backoffice php bin/console lexik:jwt:generate-keypair --overwrite
```

### 7. ✅ Vérifier que tout fonctionne

L’application Symfony est désormais accessible à l’adresse suivante :  
🔗 [http://localhost:8000](http://localhost:8000)

### 8. 🛑 Arrêter les conteneurs

Quand vous avez terminé :

```bash
docker-compose down
```

---

## 🛠️ Dépannage

### 🔁 Redémarrage complet

Si vous avez modifié le code ou les fichiers de configuration :

```bash
docker-compose down
docker-compose up --build -d
```

### 🧹 Problèmes de cache Symfony

Si vous avez des erreurs liées au cache :

```bash
docker-compose exec backoffice php bin/console cache:clear
docker-compose exec backoffice php bin/console cache:warmup
```

---

## 📬 Contact

Pour toute question ou retour concernant ce projet, vous pouvez contacter **l'équipe 9**.
