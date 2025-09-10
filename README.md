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
