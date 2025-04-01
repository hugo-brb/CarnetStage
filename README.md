# CarnetStage

## Prérequis

Avant de commencer, assurez-vous d'avoir installé les outils suivants :

-   **Docker** : [Télécharger Docker](https://www.docker.com/get-started)
-   **Docker Compose** : Si ce n'est pas inclus avec Docker, vous pouvez l'installer via [Docker Compose installation](https://docs.docker.com/compose/install/)
-   **Git** : [Télécharger Git](https://git-scm.com/)

## Étapes pour démarrer le projet

### 1. Cloner le projet

Cloner le projet à partir du dépôt Git :

```bash
git clone --recursive https://github.com/hugo-brb/CarnetStage.git
cd carnet-stage
```

### 2. Vérifier le fichier `.env`

Le fichier `.env` contient les variables d'environnement nécessaires pour configurer l'application Symfony et la base de données. Vérifiez qu'il est bien présent dans le répertoire `carnet-stage-server-24-25/`.

Si ce fichier n'est pas présent, créez-en un à partir de l'exemple suivant :

```ini
APP_ENV=dev
APP_DEBUG=true
APP_SECRET=changeme

# Configuration de la base de données (adaptée à docker-compose.yml)
DATABASE_URL="pgsql://app-stages:app-stages@db:5432/stage_db"

# Configuration des proxys et hôtes
TRUSTED_PROXIES=127.0.0.1,REMOTE_ADDR
TRUSTED_HOSTS=*

# Mailer (à configurer si besoin)
MAILER_DSN=null://null

# Configuration de CORS
CORS_ALLOW_ORIGIN=*
```

### 3. Construire et démarrer les conteneurs Docker

Dans le répertoire `carnet-stage-serveur-24-25/`, utilisez la commande suivante pour construire l'image Docker et démarrer les services définis dans `docker-compose.yml` :

```bash
docker-compose up --build -d
```

Cette commande :

-   **Construira** l'image Docker pour l'application Symfony et la base de données PostgreSQL.
-   **Lancera** les services en arrière-plan.

### 4. Installer les dépendances PHP

Une fois les conteneurs démarrés, accédez au conteneur `backoffice` et installez les dépendances PHP avec Composer :

```bash
docker-compose exec backoffice composer install --no-interaction --optimize-autoloader
```

Cela permettra de télécharger toutes les dépendances nécessaires au projet Symfony.

### 5. Appliquer les migrations de la base de données

Si la base de données n'est pas encore configurée, appliquez les migrations avec la commande suivante :

```bash
docker-compose exec backoffice php bin/console doctrine:migrations:migrate --no-interaction
```

Cela appliquera les migrations pour mettre à jour la base de données à la dernière version.

### 5.Bis Remplir la base

Executez la commande suivante :

```bash
docker-compose exec db psql -U app-stages -d stage_db -c "SELECT * FROM compte_etudiant;"
```

Si cette commande vous retourne un tableau d'utilisateurs non vide alors vous pouvez directement passer à l'étape 6, sinon cela veut dire que la base de données est incomplète, vous pouvez forcer son remplissage en exécutant la commande suivante :

```bash
docker-compose exec -T db psql -U app-stages -d stage_db < dump_INSERT.sql
```

Relancer les conteneur :
```bash
docker-compose down
docker-compose up --build -d
```

### 6. Vérifier l'application

L'application Symfony devrait maintenant être accessible sur [http://localhost:8000](http://localhost:8000).

### 7. Pour arrêter les conteneurs Docker

Lorsque vous avez fini, vous pouvez arrêter tous les conteneurs Docker avec :

```bash
docker-compose down
```

Cela arrêtera les conteneurs et supprimera les réseaux associés.

---

## Dépannage

Si vous rencontrez des problèmes avec les variables d'environnement ou des erreurs de cache Symfony, vous pouvez essayer de réinitialiser le cache :

```bash
docker-compose exec backoffice php bin/console cache:clear
docker-compose exec backoffice php bin/console cache:warmup
```

Si vous avez besoin de redémarrer les conteneurs après des modifications dans le code, utilisez :

```bash
docker-compose down
docker-compose up --build -d
```

---

## Contact

Pour toute question concernant le projet, contactez l'équipe 9.
