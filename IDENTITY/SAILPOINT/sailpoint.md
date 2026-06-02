# IdentityIQ Installation Guide

## Step 1: Install Java

1. Download and install Java Development Kit (JDK) 17 from [Oracle JDK Downloads](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html).
2. Verify the installation by running the following command in the command prompt:

    ```bash
    java --version
    ```

    Example output:

    ```
    Java 17.0.8 2023
    ```

3. If the `java` command is not recognized, add Java to the system `PATH`:

    ```bash
    echo %PATH%
    set PATH=%PATH%;"C:\Program Files\Java\jdk-17\bin"
    ```

---

## Step 2: Install Apache Tomcat

1. Download Apache Tomcat 9.0 from the [Apache Tomcat Official Website](https://tomcat.apache.org/download-90.cgi).
2. Install and extract it to:

    ```
    C:\Program Files\Apache Software Foundation\Tomcat 9.0
    ```

3. Start Tomcat and verify it's running by navigating to:

    ```
    http://localhost:8080
    ```

---

## Step 3: Deploy IdentityIQ

1. Create a folder named `identityiq` in:

    ```
    C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps
    ```
![image](https://github.com/user-attachments/assets/937ffc35-e217-4a38-91b3-1d0afcc27e6f)

2. Navigate to the IdentityIQ web application directory:

    ```bash
    cd "C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\identityiq"
    ```

3. Extract the IdentityIQ WAR file:

    ```bash
    jar -xvf identityiq.war
    ```

---

## Step 4: Set Environment Variables

### User Variables

- `JAVA_HOME` → `C:\Program Files\Java\jdk-17`
- `CATALINA_HOME` → `C:\Program Files\Apache Software Foundation\Tomcat 9.0`

### System Variables

- `JAVA_HOME` → `C:\Program Files\Java\jdk-17`
- `CATALINA_HOME` → `C:\Program Files\Apache Software Foundation\Tomcat 9.0`

---

## Step 5: Install MySQL

1. Download MySQL Server from [MySQL Downloads](https://dev.mysql.com/downloads/file/?id=536788).
2. Install MySQL and **remember the root password**.
   
![image](https://github.com/user-attachments/assets/b9dbfd76-da10-4186-92dd-96566ca867a4)
   

4. Open the MySQL Command Line Interface (CLI) and execute:

    ```sql
    source C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\identityiq\database\create_identityiq_tables-8.3.mysql
    ```

5. Verify database creation by listing databases:

    ```sql
    show databases;
    ```

    You should see `identityiq` in the list.

---

## Step 6: Configure IdentityIQ Database Connection

1. Navigate to:

    ```
    C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\identityiq\WEB-INF\classes
    ```

2. Open `iiq.properties` and modify the following lines:

    ```properties
    datasource.username=root
    datasource.password=<your_database_password>
    ```

---

## Step 7: Import IdentityIQ Artifacts

1. Navigate to:

    ```
    C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\identityiq\WEB-INF\bin
    ```

2. Run the console:

    ```bash
    .\iiq console
    ```

3. Import initialization XML:

    ```bash
    import init.xml
    ```

---

## Step 8: Access IdentityIQ

1. Open your browser and go to:

    ```
    http://localhost:8080/identityiq/login.jsf?prompt=true
    ```

2. Use the following credentials:

    - **Username:** `spadmin`
    - **Password:** `admin`

   ![image](https://github.com/user-attachments/assets/f94af4e9-6327-4c3a-929a-94bc15e5b48f)

---

## Notes

- Replace `<your_database_password>` with your actual MySQL root password.
- Make sure all services (Tomcat and MySQL) are running before accessing IdentityIQ.
