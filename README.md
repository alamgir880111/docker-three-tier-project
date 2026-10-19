Django Notes App – Dockerized Deployment

A containerized Django Notes application deployed using Docker Compose, with Nginx as a reverse proxy, Gunicorn as the WSGI application server, and MySQL 8.4 as the database.

This project demonstrates multi-container application deployment, service networking, database persistence, health checks, and automated database migrations.

🏗️ Architecture

                  User / Browser
                        |
                        | HTTP :80
                        v
                 +--------------+
                 |    Nginx     |
                 | Reverse Proxy|
                 +--------------+
                        |
                        | HTTP :8000
                        v
                 +--------------+
                 | Django App   |
                 |   Gunicorn   |
                 +--------------+
                        |
                        | MySQL :3306
                        v
                 +--------------+
                 |  MySQL 8.4   |
                 |   test_db    |
                 +--------------+
                        |
                        v
                 +--------------+
                 | mysql-data   |
                 | Docker Volume|
                 +--------------+

             Docker Network: node-app

🛠️ Technologies Used

Python

Django

Gunicorn

Nginx

MySQL 8.4

Docker

Docker Compose

Linux

Environment variables

📁 Project Structure

django-notes-app/
├── Dockerfile
├── docker-compose.yml
├── .env
├── .gitignore
├── manage.py
├── requirements.txt
├── notesapp/
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── nginx/
    ├── Dockerfile
    └── nginx.conf

Note: This is an example structure. Adjust it to match your actual project files.

⚙️ Prerequisites

Install the following tools:

Docker Engine or Docker Desktop

Docker Compose

Git (optional)

Verify the installation:

docker --version
docker compose version

🚀 Getting Started

1. Clone the Repository

git clone <YOUR_GITHUB_REPOSITORY_URL>
cd <YOUR_PROJECT_DIRECTORY>

Replace the placeholders with your repository URL and project directory.

2. Configure Environment Variables

Create a .env file in the project root. Add the Django and database settings expected by your application.

Example:

DEBUG=False
SECRET_KEY=replace-with-a-secure-secret-key
DB_NAME=test_db
DB_USER=root
DB_PASSWORD=root
DB_HOST=mysql
DB_PORT=3306

Important: These variable names are examples. Ensure they match the names used in your Django settings.py. Do not commit real credentials or production secrets to GitHub.

3. Build and Start the Application

Run this command from the directory containing docker-compose.yml:

docker compose up -d --build

Docker Compose builds the Django and Nginx images, starts MySQL, waits for its health check, and then starts the Django service. The Django command runs database migrations before launching Gunicorn.

4. Check Container Status

docker compose ps

Confirm that the nginx, db_cont, and django_cont containers are running.

5. Access the Application

Open your browser:

http://localhost

Nginx listens on host port 80 and forwards requests to the Django application through Gunicorn.

If deploying on a remote server, use the server's IP address or configured domain instead of localhost.

🐳 Docker Compose Services

Nginx

Built from the ./nginx directory.

Exposes port 80.

Acts as a reverse proxy for the Django application.

Depends on the Django container starting.

Django Application

Built from the project root.

Runs database migrations with python manage.py migrate --no-input.

Uses Gunicorn to serve the Django application on port 8000.

Loads environment variables from .env.

Connects to MySQL through the node-app Docker network.

MySQL Database

Uses the official mysql:8.4 image.

Creates the test_db database.

Uses a health check to monitor database readiness.

Stores database files in the persistent mysql-data volume.

Networking

All services use the node-app network, enabling communication between containers by service name.

For example, Django connects to MySQL using:

mysql:3306

Nginx should forward requests to the Django service using its Compose service name and port, typically:

django_app:8000

The exact upstream configuration depends on your nginx.conf.

💾 Data Persistence

The named volume mysql-data stores MySQL database files outside the container's writable layer.

This helps retain database data when containers are removed and recreated.

Stop the project without deleting the volume:

docker compose down

Warning: Running docker compose down -v removes the Compose-managed named volume and can permanently delete the database data stored in it.

🧰 Useful Docker Commands

Task

Command

Build and start

docker compose up -d --build

Check containers

docker compose ps

View all logs

docker compose logs -f

Django logs

docker compose logs -f django_app

Nginx logs

docker compose logs -f nginx

MySQL logs

docker compose logs -f mysql

Run migrations manually

docker compose exec django_app python manage.py migrate

Open Django shell

docker compose exec django_app python manage.py shell

Stop containers

docker compose down

Restart services

docker compose restart
