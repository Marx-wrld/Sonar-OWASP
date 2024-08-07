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
    environment:
      POSTGRES_USER: sonaruser
      POSTGRES_PASSWORD: velocityadmin
      POSTGRES_DB: sonarqube
    ports:
      - "5433:5432"  # Use a different host port, e.g., 5433
    volumes:
      - sonarqube_postgres_data:/var/lib/postgresql/data

  sonarqube:
    image: sonarqube:latest
    ports:
      - "9000:9000"
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://sonarqube_postgres:5432/sonarqube
      - SONAR_JDBC_USERNAME=sonaruser
      - SONAR_JDBC_PASSWORD=velocityadmin
    depends_on:
      - sonarqube_postgres
volumes:
  sonarqube_postgres_data:
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
