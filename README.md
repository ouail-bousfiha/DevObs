# DevObs

TP 1


docker network create app-network             																# creation du reseau
                                              																# admirer interface pour la base de donnée
                                                                              
docker run --name some-postgres -e POSTGRES_PASSWORD=pwd -e POSTGRES_USER=usr -e POSTGRES_DB=db -d --network=app-network postgres                                       #
docker run -d --link test:db --network=app-network -p 8080:8080 adminer

# BDD 

reseau   = app-network 
Server   = some-postgres
Username = usr
Password = pwd
Database = bd

dossier "/docker-entrypoint-initdb.d" pour executer script sql au demarrage. 								# probleme avec WSL
docker build -t oui/some-postgres . // rebuild de l'image avec "COPY initdb/ /docker-entrypoint-initdb.d" dans le Dockerfile
docker run -d --network=app-network -p 8888:5000 --name test oui/some-postgres
docker run -d --network=app-network -p 8888:5000 -v /my/own/datadir:/var/lib/postgresql/data --name test oui/some-postgres       	# persistance 

# BACKEND

Docker file hello world :

  FROM openjdk:11

  COPY Main.java /usr/src/app/
  CMD cd /usr/src/app/ ; javac Main.java
  CMD cd /usr/src/app/ ; java Main.java

docker build -t oui/test .
docker run  -p 5000:8080 --name test oui/test

application.yml :

- url: "jdbc:postgresql://some-postgres:5432/db"
- username: usr
- password: pwd

Docker file mvn :

  FROM maven:3.6.3-jdk-11 AS myapp-build
  ENV MYAPP_HOME /opt/myapp
  WORKDIR $MYAPP_HOME
  COPY pom.xml .
  COPY src ./src
  RUN mvn dependency:go-offline
  RUN mvn package -DskipTests

  FROM openjdk:11-jre
  ENV MYAPP_HOME /opt/myapp
  WORKDIR $MYAPP_HOME
  COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
  ENTRYPOINT java -jar myapp.jar

docker build -t oui/BACKEND .
docker run  -p 5000:8080 --name BACKEND --network=app-network oui/BACKEND

# http

docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > my-httpd.conf 

Dockerfile :
  FROM httpd:2.4
  COPY index.html /usr/local/apache2/htdocs/
  COPY httpd.conf /usr/local/apache2/conf/httpd.conf
  
docker build -t oui/http .
docker run -dit --name httpapp --network=app-network -p 80:80 oui/http

decommenter dans le fichier httpd.conf, mod_proxy_http et mod_proxy.

on rajouter dans le fichier httpd.conf : 

  ServerName localhost
  <VirtualHost *:80>
      ProxyPreserveHost On
      ProxyPass / http://backend:8080/
      ProxyPassReverse / http://backend:8080/
  </VirtualHost>

docker run -dit --name httpapp --network=app-network -p 8080:8080 oui/http

on cree un fichier docker compose a la racine de tout nos fichier avec les ligne suivante :

  version: '3.3'
  services:
    backend:
      build: ./API/simple-api-main/simple-api-main/simple-api
      networks: 
        - app-network
      depends_on: 
        - some-postgres 
    some-postgres: 
      build: ./bdd
      networks: 
        - app-network
    server-http:
      build: ./http 
      ports: 
        - 80:80
      networks: 
        - app-network
      depends_on: 
        - backend  
  networks:
    app-network:
    
docker-compose up --build
docker push dans nos espace en ligne
