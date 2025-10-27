# Jenkins: Deploy to Tomcat Cheatsheet

This document provides the steps to configure a Jenkins Freestyle project to build a Java web application from GitHub using Maven and automatically deploy it to a remote Tomcat server.

## 1. Prerequisite Environment Setup

### a. Java
-   Install a Java Development Kit (JDK 11 or 17 recommended).
-   Set the `JAVA_HOME` environment variable on your system.
-   Add `%JAVA_HOME%\bin` (Windows) or `$JAVA_HOME/bin` (Linux/macOS) to your system's `Path` variable.

### b. Apache Tomcat
1.  Download and unzip Apache Tomcat (e.g., version 9).
2.  **Edit `conf/server.xml`:** Change the HTTP Connector port from `8080` to `8888` to avoid conflicts with Jenkins.
    ```xml
    <Connector port="8888" protocol="HTTP/1.1" ... />
    ```
3.  **Edit `conf/tomcat-users.xml`:** Add a user with manager roles.
    ```xml
    <role rolename="manager-script"/>
    <user username="jenkins-deployer" password="password123" roles="manager-script"/>
    ```
4.  Start the Tomcat server using `bin/startup.bat` or `bin/startup.sh`.

### c. Jenkins
1.  Download `jenkins.war` and run it.
    ```bash
    java -jar jenkins.war --httpPort=8080
    ```
2.  **Install Plugins** in *Manage Jenkins > Plugins*:
    -   `Git`
    -   `Maven Integration`
    -   `Deploy to container`
3.  **Configure Maven** in *Manage Jenkins > Global Tool Configuration*:
    -   Click **Add Maven**.
    -   **Name:** `Maven3`
    -   Check **Install automatically**.
4.  **Add Tomcat Credentials** in *Manage Jenkins > Credentials*:
    -   **Kind:** Username with password
    -   **Username:** `jenkins-deployer`
    -   **Password:** `password123`
    -   **ID:** `tomcat-manager-credentials`

## 2. Application Source Code (GitHub)

Ensure your repository has the following structure and content.

-   **Project Structure:**
    ```
    .
    ├── pom.xml
    └── src
        └── main
            └── webapp
                └── index.jsp
    ```

-   **`pom.xml`:**
    ```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0" ...>
      <modelVersion>4.0.0</modelVersion>
      <groupId>com.example</groupId>
      <artifactId>my-webapp</artifactId>
      <packaging>war</packaging>
      <version>1.0-SNAPSHOT</version>
      <name>my-webapp Maven Webapp</name>
      <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
      </properties>
      <build>
        <finalName>my-app</finalName>
        <plugins>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <version>3.4.0</version>
            <configuration>
              <failOnMissingWebXml>false</failOnMissingWebXml>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </project>
    ```

-   **`src/main/webapp/index.jsp`:**
    ```jsp
    <html>
    <body>
        <h2>Hello World from Tomcat!</h2>
        <p>This page was deployed by Jenkins Build #<%= System.getenv("BUILD_NUMBER") %></p>
    </body>
    </html>
    ```

## 3. Jenkins Job Configuration

1.  **Create Job:** New Item > `Freestyle project`.
2.  **Source Code Management:**
    -   Select **Git**.
    -   **Repository URL:** Your GitHub repository URL.
    -   **Branch Specifier:** `*/main`
3.  **Build:**
    -   **Add build step:** `Invoke top-level Maven targets`.
    -   **Maven Version:** `Maven3`.
    -   **Goals:** `clean package`.
4.  **Post-build Actions:**
    -   **Add post-build action:** `Deploy WAR/EAR to a container`.
    -   **WAR/EAR files:** `target/my-app.war`
    -   **Context path:** `my-app`
    -   **Add Container:** `Tomcat 9.x Remote`.
    -   **Credentials:** `jenkins-deployer` (`tomcat-manager-credentials`).
    -   **Tomcat URL:** `http://localhost:8888`

## 4. Run and Verify

1.  Click **Build Now** in the Jenkins job.
2.  After the build succeeds, verify the deployment by visiting:
    `http://localhost:8888/my-app`
