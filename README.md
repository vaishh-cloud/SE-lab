Q1: Software Requirements Specification (SRS)
a. Abstract
This project aims to develop a centralized Campus Event Management System to streamline the organization, promotion, and participation in college events. The system provides a unified platform for students, faculty, and administrators to manage the entire event lifecycle, including event creation with detailed descriptions, user-friendly registration, a dynamic scheduling calendar, and real-time notifications for updates and reminders. The goal is to increase event attendance, enhance communication, and improve efficiency in managing campus activities.
b. Functional Requirements

Event Creation & Management: Authorized users (administrators, faculty) can create, edit, and publish events with details like title, description, date, time, location, and registration limits.
User Registration for Events: Students can view upcoming events, register (RSVP), and check their registered events.

c. Non-Functional Requirements

Usability: Intuitive interface, easy to navigate for all users with minimal training.
Performance: Responsive system with real-time notifications delivered promptly without significant delay.

d. Identification of Users

Students: Discover, register, and receive notifications about events.
Faculty: Organize and manage departmental/academic events or participate.
Administrators: Manage user accounts, oversee events, and ensure smooth operation.


Q2: Maven Java Application Development
Prerequisites

Install Java JDK 11 or higher.
Install Maven and configure JAVA_HOME.
Install Git.
Use Eclipse/IntelliJ IDE.

Steps

Clone the Repository

Run: git clone https://github.com/KumbhamBhargavi75/CampusMgmtSystem.git
Navigate: cd CampusMgmtSystem
List files: ls -R (shows pom.xml, src/, etc.)


Resolve Compilation Errors

Error: [ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.8.1:compile
Debug: Run mvn clean install -e to see details (e.g., syntax errors, JDK mismatch).
Fix: Ensure JAVA_HOME points to JDK 11+; correct code errors from log.


Resolve "Source Option 11 Not Supported"

Cause: JDK mismatch (e.g., using JDK 8).
Fix: Install JDK 11+, set JAVA_HOME, and update IDE JDK settings.
Upgrade JUnit: Remove <dependency><groupId>junit</groupId><artifactId>junit</artifactId><version>4.11</version><scope>test</scope></dependency> and rely on <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-test</artifactId><scope>test</scope></dependency> (includes JUnit 5).


Handle Missing JUnit Version

Maven uses version from spring-boot-starter-parent if omitted.
Explicitly set: <version>4.13.2</version> or use <properties><junit.version>4.13.2</junit.version></properties>.


Debug Misspelled <artifactId>

Error: [ERROR] Could not resolve dependencies (e.g., junit:junitt).
Fix: Correct typo (e.g., junit); verify via search.maven.org.


Fix <paking> Typo

Default: jar if <packaging> missing.
Impact: Builds .jar instead of .war, causing deployment failure.
Fix: Use <packaging>war</packaging>.


Add MySQL Dependency

Add: <dependency><groupId>mysql</groupId><artifactId>mysql-connector-java</artifactId><scope>runtime</scope></dependency>.


Add jQuery 3.6.1

Via WebJars: <dependency><groupId>org.webjars</groupId><artifactId>jquery</artifactId><version>3.6.1</version></dependency>
Use: <script src="/webjars/jquery/3.6.1/jquery.min.js"></script> in HTML.


Fix Invalid </artifactId>

Error: XML parsing failure, no build output.
Fix: Ensure <artifactId>my-app</artifactId> matches closing tag.



10-11. Change Build Output Directory

Add: <build><directory>${project.basedir}/build_output</directory></build>.


Add JUnit 4.13-beta-2


Update: <version>4.13-beta-2</version> in <dependency><groupId>junit</groupId><artifactId>junit</artifactId><scope>test</scope></dependency>.
Run: mvn clean test.


Push to GitHub


Run: git status, git add pom.xml, git commit -m "Updated pom", git push origin main.
Check: Visit your GitHub repo.

Run as Maven Build

In Eclipse: Right-click project > Run As > Maven build... > Goals: clean install.
In Terminal: mvn clean install.


Q3: Git & GitHub Integration with Maven Project
Commands

Initialize Git: git init
Commit Snapshot: git add .; git commit -m "Added event registration feature"
Check Status: git status
View Commit History: git log or git log --oneline
List All Branches: git branch -a
Create Branch: git checkout -b event-scheduler
Apply Patch: git apply --check name-of-the.patch; git apply name-of-the.patch
Feature Branch Workflow: git checkout -b online-feedback; git add .; git commit -m "feat: Add UI"; git add .; git commit -m "feat: Implement backend"; git checkout main; git merge online-feedback; git branch -d online-feedback
Resolve Merge Conflicts: Edit conflicted files, git add <file>, git commit
Fork & Pull Request: Fork on GitHub, git clone https://github.com/YourUsername/CampusMgmtSystem.git, git checkout -b my-new-feature, git add ., git commit -m "feat: New feature", git push origin my-new-feature, create PR on GitHub.
Compare with Remote: git fetch origin; git diff origin/main
Checkout Remote Branch: git fetch origin; git checkout event-feedback
View Last Commit Diff: git show or git diff HEAD~1 HEAD


Q4: Docker Containerization for Maven Java Application
Prerequisites

Install Docker and Docker Hub account.
Clone repo: git clone https://github.com/KumbhamBhargavi75/CampusMgmtSystem.git.
Install Maven.

Dockerfile
dockerfile# Stage 1: Build
FROM maven:3.8-jdk-11 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package

# Stage 2: Run
FROM tomcat:9.0-jdk11-openjdk-slim
RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=builder /app/target/CampusMgmtSystem.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
Commands

Check Docker Version: docker --version or docker version
Pull & Run Hello-World: docker pull hello-world; docker run hello-world
Pull & Run Ubuntu Interactively: docker pull ubuntu; docker run -it ubuntu /bin/bash
Build & Run CSIMAGE: docker build -t csimage .; docker run -d -p 8080:8080 --name campusapp csimage
Pull & Run Nginx: docker pull nginx; docker run -d -p 8888:80 --name mynginx nginx; docker ps
Start/Stop Container: docker stop mynginx; docker start mynginx
List All Containers: docker ps -a
Stop Container: docker stop campusapp
Push to Docker Hub: docker login; docker tag csimage your-dockerhub-username/csimage:latest; docker push your-dockerhub-username/csimage:latest (make public on hub.docker.com)
Check Exit Status: docker inspect campusapp (look for ExitCode) or docker logs campusapp


Q5: Docker Compose
docker-compose.yml
yamlversion: '3.8'
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
Run

Save as docker-compose.yml, then run: docker-compose up -d.
Access: http://localhost:8086 (Tomcat) or http://localhost:7007 (CEMS).
