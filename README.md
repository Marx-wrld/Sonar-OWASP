# Sonar-OWASP
Sonar &amp; OWASP security integration with github CI/CD 

Step 1. Install PostgreSQL
- Install PostgreSQL on your server or use a managed PostgreSQL service.
- Create a database and user for SonarQube.
```
sudo -u postgres psql
CREATE DATABASE sonarqube;
CREATE USER sonar WITH ENCRYPTED PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;
\q
```
Step 2. Set Up SonarQube
- Using Docker-compose
```
version: '3.8'

services:
  sonarqube_postgres:
    image: postgres:12
    container_name: sonarqube_postgres
    environment:
      POSTGRES_USER: sonaruser
      POSTGRES_PASSWORD: velocityadmin
      POSTGRES_DB: sonarqube
    ports:
      - "5433:5432"
    volumes:
      - sonarqube_postgres_data:/var/lib/postgresql/data
    networks:
      - sonarnet
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U sonaruser -d sonarqube"]
      interval: 10s
      timeout: 5s
      retries: 5

  sonarqube:
    image: sonarqube:latest
    container_name: sonarqube
    depends_on:
      - sonarqube_postgres
    ports:
      - "9000:9000"
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://sonarqube_postgres:5432/sonarqube
      SONAR_JDBC_USERNAME: sonaruser
      SONAR_JDBC_PASSWORD: velocityadmin
      ES_JAVA_OPTS: "-Xms1g -Xmx1g"
      SONARQUBE_JAVA_OPTS: "-Xms1g -Xmx2g"
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    networks:
      - sonarnet

  log_cleaner:
    image: alpine
    container_name: sonarqube_log_cleaner
    volumes:
      - sonarqube_logs:/logs
    networks:
      - sonarnet
    entrypoint: ["/bin/sh", "-c"]
    command: >
      while true; do
        echo "Cleaning SonarQube logs older than 14 days...";
        find /logs -type f -mtime +14 -exec rm -f {} \;;
        sleep 86400;
      done

volumes:
  sonarqube_postgres_data:
    driver: local
  sonarqube_data:
    driver: local
  sonarqube_extensions:
    driver: local
  sonarqube_logs:
    driver: local

networks:
  sonarnet:
    driver: bridge

```
- start the containers
```
docker-compose up -d
```
Step 3. Configure GitHub CI/CD
- In your GitHub repository, create a .github/workflows/sonarqube.yml file:
```
name: Sonar code analysis & quality

on:
  push:
    branches:
      - main


jobs:
  build:
    name: Build and analyze
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis
      - uses: sonarsource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      # If you wish to fail your job when the Quality Gate is red, uncomment the
      # following lines. This would typically be used to fail a deployment.
      # - uses: sonarsource/sonarqube-quality-gate-action@master
      #   timeout-minutes: 5
      #   env:
      #     SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```
Step 4. Access SonarQube
- Open your browser and go to http://<your_server_ip>:9000.
- Log in with default credentials (admin / admin), and change the password after your first login.

Step 5. Trigger a CI/CD Build
- Push changes to your GitHub repository. This should trigger the GitHub Actions workflow and perform a SonarQube analysis.
