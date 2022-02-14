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

Docker file mvn :

  FROM maven:3.6.3-jdk-11 AS myapp-build
  ENV MYAPP_HOME /opt/myapp
  WORKDIR $MYAPP_HOME
  COPY pom.xml .
  COPY src ./src
  RUN mvn dependency:go-offline
  RUN mvn package -DskipTests
  # Run
  FROM openjdk:11-jre
  ENV MYAPP_HOME /opt/myapp
  WORKDIR $MYAPP_HOME
  COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
  ENTRYPOINT java -jar myapp.jar

docker build -t oui/BACKEND .
docker run  -p 5000:8080 --name test oui/BACKEND

application.yml :

- url: "jdbc:postgresql://some-postgres:5432/db"
- username: usr
- password: pwd


# BACKEND
