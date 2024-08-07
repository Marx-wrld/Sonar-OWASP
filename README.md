# Sonar-OWASP
Sonar &amp; OWASP security integration with github CI/CD 

1: SONARQUBE

Using Docker-compose
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
