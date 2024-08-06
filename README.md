# Sonar-OWASP
Sonar &amp; OWASP security integration with github CI/CD 

Step 1: Add the MySQL APT GPG Key
Run the following command to fetch and add the GPG key:
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys B7B3B788A8D3785C
```
Step 2: Update Your Package Index Again
After adding the GPG key, run the update command again:
```
sudo apt update
```
Step 3: Install MySQL Server
Now you can try to install MySQL Server again:
```
sudo apt install mysql-server
```
Step 4: Create a Docker Compose File
Create a directory for your Docker setup:
```
version: '3.8'

services:
  mysql:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root_password  # Replace with your desired root password
      MYSQL_DATABASE: your_database_name    # Replace with the database name you want
      MYSQL_USER: your_username              # Replace with the MySQL user you want to create
      MYSQL_PASSWORD: user_password          # Replace with the user's password
    ports:
      - "3306:3306"
    networks:
      - my_network  # Optional: specify your network here

networks:
  my_network:  # Optional: specify your network here
```
Step 5: Create a Database
Once you have access to the MySQL shell, you can create a new database. For example, to create a database named sonarqube:

- docker exec -it 834fcbb594be mysql -u root -p
```
CREATE DATABASE sonarqube;
```
```
CREATE USER 'sonaruser'@'%' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON sonarqube.* TO 'sonaruser'@'%';
FLUSH PRIVILEGES;
``` 
Step 6: Grant Permissions
You may want to create a user for SonarQube and grant it permissions on the sonarqube database. Here's how to do that:
