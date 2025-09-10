✅ Solution for Campus Event Management System
Q1. SRS
a. Abstract

This project develops a centralized Campus Event Management System to simplify event planning, promotion, and participation in colleges.
It provides a single platform for students, faculty, and administrators to handle the event lifecycle.
Features include:

Event creation and editing

Registration for participants

Scheduling with calendar view

Real-time notifications and reminders

🎯 Goal → Improve event attendance, enhance communication, and streamline management.

b. Functional Requirements

Event Creation & Management – Authorized users (admins/faculty) can add, edit, and publish events with details (title, description, date, time, location, limits).

User Registration – Students can view upcoming events, register, and check their registered events.

c. Non-Functional Requirements

Usability – Intuitive UI for all user roles with minimal training.

Performance – Real-time notifications should be delivered with minimal latency.

d. Identification of Users

Students – Register, explore events, and receive notifications.

Faculty – Organize/manage academic or departmental events.

Administrators – Manage users, events, and ensure smooth system functioning.

Q2. Maven Java Application Development

Download Repository & List Files

git clone https://github.com/KumbhamBhargavi75/CampusMgmtSystem.git
cd CampusMgmtSystem
ls -R


Error: [ERROR] Failed to execute goal ...

Generic failure → check logs above error.

Fixes:

Correct syntax errors in .java files.

Ensure JDK 11+ is installed.

mvn clean install -e


Source option 11 not supported / Upgrade JUnit

Install JDK 11 → set JAVA_HOME.

In pom.xml remove old JUnit → replace with Spring Boot Test Starter (JUnit 5).

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>


JUnit dependency without version

If <version> missing → Maven uses dependencyManagement from parent POM.

To ensure → explicitly set:

<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.13.2</version>
  <scope>test</scope>
</dependency>


Misspelled artifactId

Maven throws:

Could not resolve dependencies: junit:junitt:jar:4.12


Debug → check pom.xml, search Maven Central, fix typos.

<paking> tag

Default → JAR packaging.

Impact → Web project fails (WAR required).

Fix:

<packaging>war</packaging>


MySQL dependency

<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <scope>runtime</scope>
</dependency>


Add jQuery 3.6.1
Option 1 – WebJars:

<dependency>
  <groupId>org.webjars</groupId>
  <artifactId>jquery</artifactId>
  <version>3.6.1</version>
</dependency>


Reference:

<script src="/webjars/jquery/3.6.1/jquery.min.js"></script>


Incorrect closing tag

Wrong: <artifactId>my-app</artfactId> → XML parse error.

Fix: <artifactId>my-app</artifactId>.

10 & 11. Change default output dir

<build>
  <directory>${project.basedir}/build_output</directory>
</build>


Add JUnit 4.13-beta-2

<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.13-beta-2</version>
  <scope>test</scope>
</dependency>


Run:

mvn clean test


Push updated project

git add pom.xml
git commit -m "Updated pom.xml with new JUnit and build config"
git push origin main

Q3. Git & GitHub Integration with Maven Project (30M)

Initialize repo: git init

Stage + commit:

git add .
git commit -m "Added event registration feature"


Check status: git status

View history: git log --oneline

Branching: git checkout -b event-scheduler

Apply patch: git apply bugfix.patch

Merge + resolve conflicts → git merge branch

Fork & PR workflow → standard GitHub flow

Compare with remote:

git fetch origin
git diff origin/main


Last commit changes: git show

Q4. Docker Containerization (20M)

Multi-stage Dockerfile

# Stage 1: Build
FROM maven:3.8-jdk-11 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package

# Stage 2: Run on Tomcat
FROM tomcat:9.0-jdk11-openjdk-slim
RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=builder /app/target/CampusMgmtSystem.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080


Build image: docker build -t csimage .

Run: docker run -d -p 8080:8080 --name campusapp csimage

Other answers:

Check images: docker images

Run hello-world: docker run hello-world

Ubuntu interactive: docker run -it ubuntu /bin/bash

Stop/start container: docker stop campusapp / docker start campusapp

Push to Docker Hub:

docker tag csimage your-dockerhub-username/csimage:latest
docker push your-dockerhub-username/csimage:latest


Logs & exit status:

docker inspect <id>
docker logs <id>

Q5. Docker Compose (10M)

docker-compose.yml

version: '3.8'

services:
  TomcatServer:
    image: tomcat:9.0-slim
    container_name: plain-tomcat
    ports:
      - "8086:8080"

  CampusMgmtSystemproject:
    image: your-dockerhub-username/csimage:latest
    container_name: campus-management-app
    ports:
      - "7007:8080"


Run both:

docker-compose up -d
