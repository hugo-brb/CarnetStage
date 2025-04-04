🇫🇷 Version française disponible [ici](README.md)

# 📘 CarnetStage

## 🚀 Prerequisites

Before getting started, make sure you have the following tools installed on your machine:

- 🐳 [**Docker**](https://www.docker.com/get-started)  
- 🐙 [**Docker Compose**](https://docs.docker.com/compose/install/) (usually included with Docker)  
- 🔧 [**Git**](https://git-scm.com/)

---

## 🌐 Accessing the Remote Server

The project has been deployed to a VPS using `docker-compose`.  
The back office is available at the following URL:

🔗 **https://156d-51-83-75-226.ngrok-free.app**

### ➕ Running Locally (without Docker)

If you want to run the back office locally without Docker (e.g., for testing), you'll need to update the `.env` file as follows:

1. **Comment out** the default line:
```env
DATABASE_URL="pgsql://app-stages:app-stages@db:5432/stage_db"
```

2. **Uncomment** this line instead:
```env
DATABASE_URL="pgsql://app-stages:app-stages@51.83.75.226:5432/stage_db?serverVersion=14&charset=utf8"
```

---

## 📱 Mobile App Connection

By default, the mobile application is configured to connect to the VPS at:  
🔗 **https://156d-51-83-75-226.ngrok-free.app**

> ⚠️ **Recommendation:**  
If you're running the back office locally via Docker, update the mobile app to point to:  
`http://10.0.2.2:8000`

This ensures that local changes affect your local database (not the one on the VPS).

🛠️ This configuration is set in the Android project:  
File `api/ApiClient.java`, at **line 34**.

---

## 🧪 Running the Project Locally

### 1. 📂 Clone the Repository

```bash
git clone --recursive https://github.com/hugo-brb/CarnetStage.git
cd CarnetStage
```

### 2. ⚙️ Check the `.env` File

In the `carnet-stage-server-24-25/` folder, ensure that the `.env` file exists and is properly configured.

If it doesn't exist, create it using the following content:

```env
APP_ENV=dev
APP_DEBUG=true
APP_SECRET=changeme

# Production Docker environment
DATABASE_URL="pgsql://app-stages:app-stages@db:5432/stage_db"

# Uncomment this for running locally (for testing, etc.)
# DATABASE_URL="pgsql://app-stages:app-stages@51.83.75.226:5432/stage_db?serverVersion=14&charset=utf8"

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

### 3. 🏗️ Build and Start Docker Containers

Make sure Docker Desktop is running, then in the `carnet-stage-server-24-25/` directory, run:

```bash
docker-compose up --build -d
```

This will:

- Build Docker images for the Symfony app and PostgreSQL
- Start the services in the background

### 4. 📦 Install PHP Dependencies

Once containers are up, install PHP dependencies using Composer:

```bash
docker-compose exec backoffice composer install --no-interaction --optimize-autoloader
```

### 5. 🧱 Apply Database Migrations

To apply the database schema:

```bash
docker-compose exec backoffice php bin/console doctrine:migrations:migrate --no-interaction
```

### 5.b 🔄 Fill the Database (if needed)

Check if the database has existing data:

```bash
docker-compose exec db psql -U app-stages -d stage_db -c "SELECT * FROM compte_etudiant;"
```

If it's empty, run the following commands to populate it:

```bash
docker-compose exec db psql -U app-stages -d stage_db -c "SET session_replication_role = 'replica';"
docker-compose exec -T db psql -U app-stages -d stage_db < dump_INSERT.sql
docker-compose exec -T db psql -U app-stages -d stage_db < dump_INSERT.sql
docker-compose exec db psql -U app-stages -d stage_db -c "SET session_replication_role = 'origin';"
```

### 6. 🔐 Generate JWT Key Pair

To enable authentication with the mobile app:

```bash
docker exec backoffice php bin/console lexik:jwt:generate-keypair --overwrite
```

### 7. ✅ Test the Application

The Symfony app should now be available at:  
🔗 [http://localhost:8000](http://localhost:8000)

### 8. 🛑 Stop the Containers

To stop the application:

```bash
docker-compose down
```

This will shut down containers and remove the associated Docker network.

---

## 🛠️ Troubleshooting

### 🔁 Restart Everything

If you’ve made code changes or modified configs:

```bash
docker-compose down
docker-compose up --build -d
```

### 🧹 Clear Symfony Cache

If you're facing issues related to cache:

```bash
docker-compose exec backoffice php bin/console cache:clear
docker-compose exec backoffice php bin/console cache:warmup
```

---

## 📬 Contact

For any questions or feedback, feel free to reach out to **Team 9**.
