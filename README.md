0) One-time setup (before the task)

Install JDK 21 (Temurin or Oracle).

Verify (optional, outside Eclipse):
java -version → should say 21.

Install Eclipse IDE for Enterprise Java and Web Developers (includes Maven + Tomcat tooling).

Add JDK 21 to Eclipse

Window > Preferences > Java > Installed JREs > Add… > Standard VM

Point to your JDK 21 folder. Check it as default.

(Optional) Tomcat

If you’ll run it locally: download Tomcat 10.1+ (Jakarta Servlet 6). We’ll match the servlet dependency to this.

1) Clone the repo & see Tomcat/Maven issues (2M)

In Eclipse

File > Import… > Git > Projects from Git (with URI) > Next

URI: https://github.com/KumbhamBhargav/HospitalMSystem.git
Next through branch & local path screens.

Choose “Import as general project” → then Eclipse will detect pom.xml and offer “Import as existing Maven project” (or you can do: Right-click project > Configure > Convert to Maven Project).

Check the layout

You should see:
pom.xml, src/main/java, src/main/webapp (typical Maven webapp layout).

See problems quickly

Open Window > Show View > Problems.

Right-click project → Run As > Maven build… → Goals: validate → **Run`.

Common errors you might see now:

“package jakarta.servlet does not exist” (missing servlet API or wrong one).

“Packaging is ‘jar’ but contains web resources” (needs war packaging).

Java version mismatch (Maven not using 21).

2) Clean & package; basic troubleshooting (3M)

Right-click project → Run As > Maven build… → Goals: clean package → Run.

If dependencies fail to download or build errors persist:

Right-click project → Maven > Update Project… (Alt+F5) → Force Update of Snapshots/Releases → OK.

If you still see Java version errors, complete step 3 (compiler plugin + JDK 21).

3) Update pom.xml: JDK 21 + Servlet API (3M)

Open pom.xml (Eclipse has a nice editor). Make these changes:

a) Ensure it builds a WAR
<packaging>war</packaging>

b) Compiler for Java 21

Use the compiler plugin (best practice; beats only setting properties):

<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.11.0</version>
      <configuration>
        <release>21</release>
      </configuration>
    </plugin>
  </plugins>
</build>


If you already have a <plugins> block, just add the plugin inside it.

c) Servlet API dependency (match your Tomcat!)

If using Tomcat 10.1+ (recommended with JDK 21) → Jakarta Servlet 6:

<dependencies>
  <dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.0.0</version>
    <scope>provided</scope>
  </dependency>
  <!-- other project deps -->
</dependencies>


If using Tomcat 9.x → javax line instead:

<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>4.0.1</version>
  <scope>provided</scope>
</dependency>


Why scope = provided? The container (Tomcat) supplies the servlet classes at runtime; you don’t want to bundle them inside your WAR.

Save pom.xml. Eclipse will auto-download dependencies.

Now try: Right-click project → Run As > Maven build… → Goals: clean compile → **Run`.

If it fails, the usual reasons are:

You’re on Tomcat 10.1 but added javax (should be jakarta).

Maven still compiling with an older JDK. Fix by ensuring:
Window > Preferences > Maven > Installations → pick the Maven that uses your JDK 21 (or let m2e use embedded but ensure Eclipse default JRE is JDK 21). Then Project > Clean…, and Maven > Update Project… again.

4) Build the project again (3M)

Run the full build:

Right-click project → Run As > Maven build… → Goals: clean package → **Run`.

If you still get source/target mismatch errors, keep the <release>21</release> plugin as above (do not mix <source>/<target> and <release> in the same config).

5) Bad dependency coordinates scenario (3M)

If your <dependencies> has something like:

<groupId>SE</groupId>
<artifactId>...</artifactId>
<version>6.0</version>


Maven will fail to resolve it (artifact not found).
Fix: Replace with real coordinates (e.g., jakarta.servlet:jakarta.servlet-api:6.0.0 or javax.servlet:javax.servlet-api:4.0.1) as shown above.

6) WAR name, SNAPSHOT & explaining the generated file (6a + 6b) (6M)
a) WAR file name (HospitalMSystem vs HospitalMgmtSystem)

By default, Maven names the WAR as:

target/<artifactId>-<version>.war


If your <artifactId> is HospitalMSystem and <version> is 1.0-SNAPSHOT,
you’ll get target/HospitalMSystem-1.0-SNAPSHOT.war.

To force a pretty name (e.g., HospitalMgmtSystem.war), add:

<build>
  <finalName>HospitalMgmtSystem</finalName>
  <!-- keep your plugins here too -->
</build>


Rebuild (clean package) → you’ll get:

target/HospitalMgmtSystem.war

b) Remove SNAPSHOT

Change:

<version>1.0</version>


(or keep finalName as above; either approach works).

c) What is inside the WAR (explain the generated file)

A WAR is a zip of your web app:

/WEB-INF/classes → your compiled .class files

/WEB-INF/lib → your runtime libs (except those with scope=provided, e.g., servlet API)

JSPs/HTML/CSS/JS from src/main/webapp

WEB-INF/web.xml (if present)

You deploy this as-is to Tomcat: copy to tomcat/webapps or add via Eclipse Servers view.

(Optional) Run on Tomcat from Eclipse (to “show issues in Tomcat”)

Window > Show View > Servers → No servers are available. Click this link to create a new server.

Apache > Tomcat v10.1 Server → point to your Tomcat install folder. Set JRE to JDK 21.

Add your project to the server → Finish.

Right-click server → Start.

If you see errors like:

ClassNotFoundException: jakarta.servlet. → you used javax coords on Tomcat 10+. Switch to jakarta.servlet-api:6.0.0.

ClassNotFoundException: javax.servlet. → you used jakarta on Tomcat 9. Switch to javax.servlet-api:4.0.1.

HTTP 404 → check context path, servlet mappings, or welcome file.

Quick reference: minimal pom.xml bits you’ll likely need
<project>
  <!-- … groupId, artifactId, version … -->
  <packaging>war</packaging>

  <properties>
    <maven.compiler.release>21</maven.compiler.release>
  </properties>

  <dependencies>
    <!-- Choose ONE of these blocks depending on Tomcat version -->

    <!-- Tomcat 10.1+ (Jakarta Servlet 6) -->
    <dependency>
      <groupId>jakarta.servlet</groupId>
      <artifactId>jakarta.servlet-api</artifactId>
      <version>6.0.0</version>
      <scope>provided</scope>
    </dependency>

    <!-- OR Tomcat 9.x (Servlet 4, javax) -->
    <!--
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
      <scope>provided</scope>
    </dependency>
    -->
  </dependencies>

  <build>
    <finalName>HospitalMgmtSystem</finalName>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.11.0</version>
        <configuration>
          <release>21</release>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>



Q3. Git and GitHub Integration with Maven Project – 30M
Task: Using the Maven project above, perform Git tasks (1-13, marks as noted). Assume repo is cloned; initialize if needed.
Guidance (Commands in terminal; use git status often to verify):

Made Changes to One File but Not Staged; Realize They Are Unwanted (2M): Discard: git checkout -- <filename> (or git restore <filename> in Git 2.23+).
Made a Commit with Wrong Message; Haven't Pushed. How to Fix (2M): Amend: git commit --amend -m "Correct message".
Fix: View Commit History in Readable Format (2M): git log --oneline --graph (shows branches/commits visually).
Create Branch "Feature/patient", Switch to It (2M): git checkout -b Feature/patient (creates and switches).
Made 2 Commits Locally; Don't Want to Upload to Remote. What Do (2M): Don't push: git push origin Feature/patient only when ready. To undo unpushed: git reset --hard HEAD~2.
Create Branch "Suggestions", Merge with Both Local/Remote? How (3M): Create: git checkout -b Suggestions. Merge local: git merge Feature/patient. Merge remote: git fetch origin; git merge origin/main.
Pull Latest Changes from Remote, Merge to Local (2M): git pull origin main (fetches and merges).
Specific Remote Repo; Need to Push Changes to New Branch Without Losing Local (2M): Create branch: git checkout -b new-branch. Push: git push -u origin new-branch (sets upstream without affecting local).
After Running Git Pull, Notice Local Branch Behind Remote. What Happened (2M): Remote had new commits; pull merged them. Fix conflicts if any: Edit files, git add ., git commit.
Pushed to Patient Branch on Remote; Contains Local Changes. How Would This Affect (3M): Remote now has your changes; team can pull. No loss if committed properly.
From Remote Apply Patch File Provided by Teammate, Include in Commit History (3M): Apply: git apply patchfile.patch. Ensure in commit: git add .; git commit -m "Applied patch". Command: git apply for patch application.
Teammate Emailed Patch with CSS Fix; Apply Locally, Include in Commit (5M): git apply css-fix.patch. Test, then git add .; git commit -m "Fixed CSS via patch". Ensures it's part of history for tracking.
Which Git Command Used to Apply Patch and Ensure Part of Commit History (3M): git apply <patchfile> to apply; git commit to include in history.

Practice Tip: Use a test repo. Simulate with git init if main repo unavailable. Document git log outputs. Total Q3: 30M – Emphasize branching/merging (15M) and troubleshooting (15M).
Q4. Docker Containerization for Maven Application – 20M (10+2+8)
Task: Containerize the Maven project. Refer to given link/documentation for Dockerfile. Ensure copies WAR/JAR and runs on Tomcat base image. (Subparts 1-10 for CLI commands.)
Guidance (Since link https://github.com/archanareddyse/labinternal-1.git has no specific Dockerfile, use standard Tomcat example):

Write Dockerfile (Refer Link/Doc): Create Dockerfile in project root:
dockerfileFROM tomcat:9.0-jdk11-openjdk  # Base Tomcat image
COPY target/HospitalMgmtSystem.war /usr/local/tomcat/webapps/  # Copy WAR
EXPOSE 8080  # Expose port
CMD ["catalina.sh", "run"]  # Run Tomcat



Command to Run Official Nginx Image as Container Named "my_nginx" (1M): docker run --name my_nginx -d -p 8080:80 nginx:latest.
CLI Command to Docker Build (1M): docker build -t hospital-mgmt . (in project dir with Dockerfile).
Command to Run Built Image, List All Docker Images with Name "hospital-mgmt" (1M): Run: docker run --name hospital-container -p 8080:8080 -d hospital-mgmt. List: docker images | grep hospital-mgmt.
Pull Run Command Map Container Port 8080 to Host Port 9090 (1M): docker run --name hospital-app -p 9090:8080 -d hospital-mgmt.
After Testing, Verify If Accessible in Container and List Containers (1M): Access: docker exec -it hospital-app bash (inside, check ls /usr/local/tomcat/webapps). List: docker ps -a.
While Running Container, Tag and Push to Docker Hub Account (1M): Tag: docker tag hospital-mgmt yourusername/hospital-mgmt:latest. Push: docker login; docker push yourusername/hospital-mgmt:latest.
From Container Browser Steps, What to Do to Stop Mapping Now? Access App? (1M): Stop: docker stop hospital-app. Access: Restart with new port, e.g., docker run -p 8080:8080 ....
Push Custom Image to Inspect Logs, Debug Issue (1M): docker logs hospital-app (for errors like port bind fail).
Make Available Publicly with Deployed App to Docker Hub Account (1M): After push (step 6), share repo URL or docker pull yourusername/hospital-mgmt.
CLI to List Removed Containers Open Shell Interactively to Check (1M): List: docker ps -a. Shell: docker exec -it <container_id> /bin/bash.

Practice Tip: Build WAR first (mvn package), then Docker. Test run: http://localhost:8080/HospitalMgmtSystem. Total Q4: 20M – Dockerfile (10M), Commands (10M).
Q5. Docker Compose – 10M
Task: Write docker-compose.yml for multi-container: Container 1 – Use pushed Docker image. Container 2 – MongoDB as DB, ensure both run together. Demonstrate startup.
Guidance:

Create docker-compose.yml:
yamlversion: '3.8'
services:
  app:
    image: yourusername/hospital-mgmt:latest  # Pushed image
    ports:
      - "8080:8080"
    depends_on:
      - mongo
    environment:
      - DB_HOST=mongo
      - DB_PORT=27017
  mongo:
    image: mongo:latest
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=pass

Run: docker compose up -d. Verify: docker compose ps (both up), http://localhost:8080. Stop: docker compose down. Insert data: docker exec -it <mongo_container> mongosh (e.g., db.patients.insertOne({name: "Test"})).

Practice Tip: Ensure app connects to MongoDB via env vars. Total Q5: 10M – YAML (5M), Demo (5M).
Q6. Git Pull with Remote Changes – 4M
Task: Explain process to bring local branch up-to-date without losing changes from https://github.com/KumbhamBhargav/HospitalMSystem.
Guidance: git fetch origin; git merge origin/main (or git pull origin main). If conflicts: Edit files, git add ., git commit. Without losing: Stash changes first (git stash; git pull; git stash pop).
Q7. Add Central Dependency Java Servlet API – 6M
Task: Add to existing project, check pom.xml, run.
Guidance: Add to <dependencies>:</dependencies>
xml<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>  <!-- For Tomcat -->
</dependency>
Run mvn clean package. Check: No compilation errors for servlets.
Q8. Developer Removes <dependencies> Section Completely – 6M</dependencies>
Task: Will Maven Build? Issues During Testing?
Guidance: Maven will fail (e.g., "No compiler plugin" or missing libs). Issues: Compilation fails (JDK errors), no WAR generated. Fix: Restore section or add minimal deps.
Q9. Failed to Execute Goal org.apache.maven.plugins:maven-compiler-plugin:3.1 – 6M
Task: Error [ERROR] No compiler, [INFO] Compilation failure. What Issues?
Guidance: Issues: Missing/outdated compiler plugin, JDK mismatch. Fix: Add/update plugin as in Q2.3.
Q10. Running Has JAR Name as JDKName:localerror:80FoodSys/finalName – 3M
Task: In Build, What Is finalName?
Guidance: finalName overrides artifactId-version for output (e.g., <finalname>HospitalMgmtSystem</finalname> creates HospitalMgmtSystem.war).
Q11. Your Project Meant to Deploy on Tomcat, but <packaging>jar</packaging> – 3M
Task: Like What for WAR?
Guidance: Change to <packaging>war</packaging> in pom.xml for Tomcat deployment.
Q12. In pom.xml, Write curl - What? (3M)
Task: Write curl to download pom.xml.
Guidance: curl -O https://raw.githubusercontent.com/KumbhamBhargav/HospitalMSystem/main/pom.xml.
Q13. Will Maven Accept pom.xml, Push to GitHub of Your Account (3M)
Task: Complete pom.xml, push.
Guidance: Yes, if valid XML. Push: git add pom.xml; git commit -m "Updated pom"; git push origin main.
