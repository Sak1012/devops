# devops

### 1. **Steps to Create a Maven Build Pipeline in Azure Portal**

#### Aim:

To create a Maven build pipeline in Azure DevOps to automate the build process for a Java project, including compilation, testing, and packaging of the project.

#### Procedure:

1. **Sign in to Azure DevOps Portal**:

   - Open the Azure DevOps portal ([dev.azure.com](https://dev.azure.com/)).
   - Sign in with your Azure credentials.

2. **Create a New Project**:

   - Navigate to "Azure DevOps" and click on "Create Project."
   - Provide a name for the project (e.g., `MavenPipelineProject`), select the visibility (Public/Private), and click on "Create."
   - The new project will be created, and you'll be directed to the project dashboard.

3. **Set Up a Git Repository**:

   - Go to "Repos" → "Files" → "Initialize repository" if you don't have a repository already.
   - If you have a remote repository (e.g., GitHub or Bitbucket), click on "Import a repository" and provide the Git URL.
   - Your Maven project should have the following structure:
     ```
     /src
       /main
         /java
     /pom.xml
     ```

4. **Navigate to Pipelines**:

   - Go to "Pipelines" → "Create Pipeline."
   - Choose where your code is hosted: Select GitHub, Azure Repos, or another Git service. If you have integrated GitHub, log in to authorize access.
   - After selecting the repository, choose the pipeline configuration.

5. **Choose Maven Template**:

   - Azure will detect the `pom.xml` file in your Maven project and offer the Maven build template.
   - You can use either the YAML or Classic Editor. For this example, choose the **YAML** option for better flexibility and future scalability.

6. **Configure YAML Pipeline**:

   - Azure will automatically generate a YAML file based on the selected Maven template.
   - Modify the pipeline file as per your project requirements. Here's an example YAML pipeline configuration:

     ```yaml
     trigger:
       branches:
         include:
           - main

     pool:
       vmImage: "ubuntu-latest"

     steps:
       - task: Maven@3
         inputs:
           mavenPomFile: "pom.xml"
           goals: "clean install"
           options: "-Xmx1024m"
           publishJUnitResults: true
           testResultsFiles: "**/surefire-reports/TEST-*.xml"

       - task: PublishTestResults@2
         inputs:
           testResultsFormat: "JUnit"
           testResultsFiles: "**/surefire-reports/*.xml"
           failTaskOnFailedTests: true
     ```

   **Explanation**:

   - **trigger**: Defines that the pipeline will trigger automatically for commits pushed to the `main` branch.
   - **pool**: Specifies the virtual machine (VM) image to use during the build process. In this case, it's Ubuntu.
   - **Maven@3**: Uses the Maven build task. It references the `pom.xml` file and runs the `clean install` goal, which compiles and packages the Java application.
   - **options**: Allows you to specify additional JVM memory parameters if needed.
   - **PublishTestResults@2**: Publishes the test results generated during the Maven build. The format is JUnit, and the results are collected from `surefire-reports`.

7. **Save and Run**:

   - Save the pipeline configuration. Azure DevOps will automatically create and queue the pipeline for execution.
   - The pipeline will start running, executing Maven commands (`clean`, `compile`, `install`) and running the unit tests defined in the project.

8. **Monitor Pipeline Execution**:

   - As the pipeline runs, you can monitor the logs in real time. This will show each step, including build success, test results, and any errors that occur.

9. **View Build Artifacts and Test Results**:
   - After the pipeline completes, the build artifacts (such as `.jar` or `.war` files) will be available for download.
   - You can also view detailed test results by going to the "Tests" tab in the pipeline summary.

#### Example Maven `pom.xml` Configuration:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- JUnit dependency for testing -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Maven Surefire Plugin for running unit tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0-M5</version>
            </plugin>
        </plugins>
    </build>
</project>
```

#### Result:

A Maven build pipeline is successfully created in Azure DevOps, which triggers automatically for every push to the repository. The pipeline compiles the Java code, runs tests, and packages the project.

---

### 2. **Commands and Configurations for Running Regression Tests in a Maven Build Pipeline in Azure DevOps**

#### Aim:

To set up and run regression tests in the Maven build pipeline in Azure DevOps, ensuring code quality through automated testing.

#### Procedure:

1. **Configure Test Dependencies**:

   - In your `pom.xml`, include necessary testing dependencies. This might include frameworks like JUnit or TestNG for testing, as well as libraries like Mockito for mocking.

   Example `pom.xml` with dependencies:

   ```xml
   <dependencies>
       <!-- JUnit for unit and regression testing -->
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>4.13.2</version>
           <scope>test</scope>
       </dependency>

       <!-- Mockito for unit testing -->
       <dependency>
           <groupId>org.mockito</groupId>
           <artifactId>mockito-core</artifactId>
           <version>3.9.0</version>
           <scope>test</scope>
       </dependency>
   </dependencies>
   ```

2. **Modify the Maven Build Pipeline for Testing**:

   - Update your pipeline's YAML file to execute the Maven `verify` phase, which includes running tests and ensuring their correctness.

   Example YAML Configuration for Regression Testing:

   ```yaml
   trigger:
     branches:
       include:
         - main

   pool:
     vmImage: "ubuntu-latest"

   steps:
     - task: Maven@3
       inputs:
         mavenPomFile: "pom.xml"
         goals: "clean verify"
         options: "-DskipTests=false -Dmaven.test.failure.ignore=true"
         javaHomeOption: "JDKVersion"
         jdkVersionOption: "1.11"
         mavenVersionOption: "Default"

     - task: PublishTestResults@2
       inputs:
         testResultsFormat: "JUnit"
         testResultsFiles: "**/target/surefire-reports/TEST-*.xml"
         failTaskOnFailedTests: false
   ```

   **Explanation**:

   - The goal `verify` is used here, which runs the `integration-test` and `test` phases of the Maven lifecycle, ensuring all tests (including regression tests) are executed.
   - The `options` flag is used to ensure that tests are not skipped (`-DskipTests=false`) and that the pipeline doesn't fail if some tests fail (`-Dmaven.test.failure.ignore=true`).
   - **PublishTestResults@2** is used to collect the test results generated by Maven, typically located in `surefire-reports`.

3. **Set Up Test Categories (Optional)**:

   - If you have different types of tests (unit, integration, regression), you can use Maven profiles to separate them. For example, define a regression profile in the `pom.xml`:
     ```xml
     <profiles>
         <profile>
             <id>regression-tests</id>
             <activation>
                 <property>
                     <name>!skipTests</name>
                 </property>
             </activation>
             <build>
                 <plugins>
                     <plugin>
                         <groupId>org.apache.maven.plugins</groupId>
                         <artifactId>maven-surefire-plugin</artifactId>
                         <version>3.0.0-M5</version>
                         <configuration>
                             <includes>
                                 <include>**/regression/**</include>
                             </includes>
                         </configuration>
                     </plugin>
                 </plugins>
             </build>
         </profile>
     </profiles>
     ```

   This configuration ensures that only the regression tests located under the `regression` directory are run when the `regression-tests` profile is activated.

4. **Run Regression Tests in the Pipeline**:

   - You can modify your pipeline to use the regression profile during the build:
     ```yaml
     steps:
       - task: Maven@3
         inputs:
           mavenPomFile: "pom.xml"
           goals: "clean verify -P regression-tests"
           options: "-DskipTests=false"
     ```

5. **Save and Run**:

   - Save the pipeline configuration and queue a new build. Azure DevOps will execute the pipeline, running the regression tests as part of the build process.

6. **View Test Results**:
   - After the pipeline finishes, the test results will be available for review in the "Tests" tab, where you can inspect the pass/fail status of individual test cases.

#### Result:

A Maven pipeline is successfully set up to run regression tests in Azure DevOps. Test results are published and reviewed as part of the CI/CD process.

---

### 3. **Procedure for Installing Jenkins in the Azure Cloud Environment**

#### Aim:

To install Jenkins in the Azure cloud environment and configure it for continuous integration and deployment (CI/CD).

#### Procedure:

1. **Create a Virtual Machine (VM) in Azure**:

   - Sign in to the [Azure portal](https://portal.azure.com/).
   - Navigate to **Create a resource** and select **Virtual Machine** under the Compute section.
   - Fill in the required details:
     - **Resource Group**: Create or select an existing resource group.
     - **VM Name**: Example, `JenkinsServer`.
     - **Region**: Choose the appropriate region for your deployment.
     - **Image**: Select **Ubuntu Server 20.04 LTS** or the latest supported version.
     - **Size**: Choose an instance type depending on the size of your workload (e.g., `Standard_DS1_v2` for moderate workloads).
   - Under **Administrator Account**, choose a method to sign in (SSH or Password).
   - Allow **HTTP**, **HTTPS**, and **SSH (port 22)** traffic under **Inbound port rules**.
   - Click **Review + Create**, then **Create** after the review.

2. **Access the Virtual Machine**:

   - Once the VM is created, navigate to it and click **Connect** → **SSH**.
   - Use the SSH command provided in the portal to connect to the VM from your local machine.

   ```bash
   ssh azureuser@<VM-IP-Address>
   ```

3. **Update the VM and Install Java**:

   - Update the package index to ensure the latest versions of software are available:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```
   - Install Java (Jenkins requires Java to run):
     ```bash
     sudo apt install openjdk-11-jdk -y
     ```
   - Verify the Java installation:
     ```bash
     java -version
     ```

4. **Add Jenkins Repository**:

   - Add the Jenkins repository key:
     ```bash
     curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
     /usr/share/keyrings/jenkins-keyring.asc > /dev/null
     ```
   - Append the Debian package repository address to your sources list:
     ```bash
     echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
     https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null
     ```

5. **Install Jenkins**:

   - Update the package list and install Jenkins:
     ```bash
     sudo apt update
     sudo apt install jenkins -y
     ```
   - Start Jenkins:
     ```bash
     sudo systemctl start jenkins
     ```
   - Enable Jenkins to start at boot:
     ```bash
     sudo systemctl enable jenkins
     ```

6. **Configure Firewall**:

   - Open port 8080 (the default Jenkins port) to allow traffic:
     ```bash
     sudo ufw allow 8080
     sudo ufw allow OpenSSH
     sudo ufw enable
     ```

7. **Access Jenkins Web Interface**:

   - In your browser, go to `http://<VM-IP-Address>:8080`.
   - The Jenkins unlock page will appear. To retrieve the initial admin password, run:
     ```bash
     sudo cat /var/lib/jenkins/secrets/initialAdminPassword
     ```
   - Copy the password, paste it into the Jenkins unlock page, and click **Continue**.

8. **Install Suggested Plugins**:

   - On the next screen, click **Install Suggested Plugins**.
   - Jenkins will automatically install common plugins such as Git, Maven, and Gradle.

9. **Create an Admin User**:

   - Once the plugins are installed, you will be prompted to create the first admin user.
   - Provide the necessary details and click **Save and Finish**.

10. **Start Using Jenkins**:
    - After the setup is complete, you will be redirected to the Jenkins dashboard, ready to create your first project.

#### Result:

Jenkins is successfully installed on an Azure VM. You can now access Jenkins through the browser and use it for setting up CI/CD pipelines.

---

### 4. **Sequential Steps for Creating a CI Pipeline Using Jenkins for a Java Project**

#### Aim:

To create a Continuous Integration (CI) pipeline in Jenkins tailored for a Java project to automatically build, test, and package the application.

#### Procedure:

1. **Open Jenkins Dashboard**:

   - Access Jenkins via `http://<VM-IP-Address>:8080` and log in using your admin credentials.

2. **Install Required Plugins**:

   - Go to **Manage Jenkins** → **Manage Plugins**.
   - Under the **Available** tab, search for and install:
     - **Git plugin** (for Git repository integration).
     - **Maven Integration plugin** (for Maven-based Java projects).
   - After installing the plugins, restart Jenkins if prompted.

3. **Create a New Jenkins Job**:

   - From the Jenkins dashboard, click **New Item**.
   - Enter the job name (e.g., `JavaCIProject`), select **Freestyle project**, and click **OK**.

4. **Set Up Source Code Management (SCM)**:

   - Under the job configuration, go to the **Source Code Management** section.
   - Choose **Git**.
   - Enter the **Repository URL** of your Java project (e.g., `https://github.com/user/repo.git`).
   - If the repository is private, provide the necessary **credentials**.
   - Specify the **branch** to build (e.g., `*/main`).

5. **Configure Build Triggers**:

   - Scroll down to the **Build Triggers** section.
   - Check the option **Poll SCM** and set the schedule (e.g., `H/5 * * * *`) to poll the repository every 5 minutes for changes.
     - `H/5`: Polls the repository every 5 minutes.
     - `* * * *`: Represents minute, hour, day, month, and day of the week, respectively.

6. **Add Build Steps**:

   - In the **Build** section, click **Add build step** → **Invoke top-level Maven targets**.
   - Configure the following:
     - **Goals**: Enter `clean install` to compile, test, and package the project.
     - **POM**: Provide the path to the `pom.xml` file (e.g., `my-app/pom.xml`).

   Example Maven Command:

   ```bash
   clean install
   ```

7. **Post-Build Actions**:

   - Click **Add Post-Build Action** → **Publish JUnit test result report**.
   - Enter the path to the JUnit report files (e.g., `**/target/surefire-reports/TEST-*.xml`).
   - This step ensures that test results are published after each build.

8. **Save and Build**:

   - Click **Save** to finalize the job configuration.
   - Manually trigger a build by clicking **Build Now** from the job dashboard.

9. **Monitor Build Progress**:

   - Go to the **Build History** section on the left panel to monitor the build.
   - Jenkins will fetch the latest code from Git, build the project using Maven, run the tests, and package the application.
   - You can view the console output to trace each step of the build process.

10. **Set Up Notifications (Optional)**:
    - If you want to receive notifications upon build success or failure, install the **Email Extension Plugin**.
    - In the job configuration, add a post-build action **Editable Email Notification**.
    - Configure email recipients and set triggers for failure/success notifications.

#### Example Jenkins Job Configuration:

```bash
Maven goals: clean install
SCM Polling: H/5 * * * *
Post-Build Action: Publish JUnit report
JUnit report path: **/target/surefire-reports/TEST-*.xml
```

#### Result:

A CI pipeline is successfully created in Jenkins for a Java project. Jenkins will automatically fetch the code, build the project using Maven, run tests, and publish the results for every commit pushed to the repository.

---

### 5. **Procedure for Setting Up a Continuous Deployment (CD) Pipeline in Jenkins and Deploying It to Azure**

#### Aim:

To set up a Continuous Deployment (CD) pipeline in Jenkins for automating the deployment of a Java application to the Azure cloud environment.

#### Procedure:

1. **Prerequisites**:

   - Make sure Jenkins is already set up with the necessary plugins installed:
     - **Azure CLI Plugin**: Used for deploying resources to Azure.
     - **Azure Web App Plugin**: For Azure web app integration.
     - **Maven Plugin** (if using Maven for the Java build).

2. **Create an Azure Web App**:

   - In the [Azure Portal](https://portal.azure.com/), create an Azure Web App that will serve as the deployment target.
     - Navigate to **Create a resource** → **Web App**.
     - Choose the appropriate **Resource Group**, **App Service Plan**, and **Runtime stack** (Java 11, for example).
     - Once created, take note of the **App Service URL** as it will be needed for deployment.

3. **Install Azure CLI on the Jenkins Machine**:

   - If Azure CLI is not installed, SSH into the Jenkins machine and run the following commands:
     ```bash
     curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
     az login
     ```
   - Authenticate with your Azure credentials.

4. **Create a New Jenkins Pipeline Job**:

   - In the Jenkins dashboard, click **New Item**.
   - Choose **Pipeline** as the project type, give it a name like `CDJavaPipeline`, and click **OK**.

5. **Configure Source Code Management**:

   - In the **Pipeline** configuration page, under **Pipeline** → **Definition**, select **Pipeline script from SCM**.
   - Choose **Git** as the SCM and provide the repository URL (e.g., `https://github.com/user/repo.git`).
   - Specify the branch (e.g., `*/main`).

6. **Write the Jenkins Pipeline Script**:

   - In the **Pipeline** section, write a pipeline script that will:
     - Build the application using Maven.
     - Deploy the package to Azure using Azure CLI.

   Example Jenkinsfile:

   ```groovy
   pipeline {
       agent any

       stages {
           stage('Build') {
               steps {
                   // Build the project using Maven
                   sh 'mvn clean package'
               }
           }

           stage('Deploy to Azure') {
               steps {
                   // Deploy the JAR to Azure Web App
                   withCredentials([string(credentialsId: 'azure-credentials', variable: 'AZURE_CREDENTIALS')]) {
                       sh """
                       az login --service-principal -u $AZURE_CREDENTIALS --tenant <tenant-id> --password <password>
                       az webapp deployment source config-zip --resource-group <your-resource-group> --name <your-app-name> --src target/your-app.jar
                       """
                   }
               }
           }
       }
   }
   ```

   In this script:

   - The `Build` stage uses Maven to compile the Java code and package it into a JAR file.
   - The `Deploy to Azure` stage uses Azure CLI to deploy the JAR to the specified Azure Web App.

7. **Configure Azure Credentials in Jenkins**:

   - Go to **Manage Jenkins** → **Manage Credentials**.
   - Add a new **Azure Service Principal** credential that contains your Azure subscription ID, tenant ID, and service principal details.
   - Reference these credentials in the `withCredentials` block in the Jenkins pipeline script.

8. **Trigger the Pipeline**:

   - After saving the pipeline configuration, manually trigger the pipeline by clicking **Build Now**.
   - Jenkins will:
     - Pull the latest code from Git.
     - Build the application using Maven.
     - Deploy the built JAR file to the Azure Web App.

9. **Monitor Deployment in Azure**:
   - After the build and deployment stages are complete, visit the Azure Portal to monitor the Azure Web App deployment.
   - You can access the deployed application by visiting the Web App's public URL.

#### Result:

A CD pipeline is successfully set up in Jenkins, automatically deploying the Java application to the Azure Web App every time new code is pushed to the repository.

---

### 6. **Structuring an Ansible Playbook to Establish Basic Web Application Infrastructure in a Lab Environment**

#### Aim:

To create an Ansible playbook that provisions a basic web application infrastructure (including a web server, firewall configuration, and application setup) in a lab environment.

#### Procedure:

1. **Install Ansible**:

   - SSH into the control node (your local machine or a cloud VM) and install Ansible:
     ```bash
     sudo apt update
     sudo apt install ansible -y
     ```

2. **Create an Inventory File**:

   - Create a file named `hosts` to list the target servers where the web application will be deployed:
     ```ini
     [webservers]
     192.168.1.10 ansible_user=azureuser
     ```

3. **Write the Ansible Playbook**:

   - Create an Ansible playbook named `setup-web.yml` that provisions the infrastructure for a basic web application. The playbook will:
     - Install the necessary packages (Apache, for example).
     - Start and enable the web server.
     - Configure firewall rules.

   Example Playbook:

   ```yaml
   ---
   - hosts: webservers
     become: yes

     tasks:
       - name: Install Apache Web Server
         apt:
           name: apache2
           state: present
           update_cache: yes

       - name: Start Apache Service
         service:
           name: apache2
           state: started
           enabled: yes

       - name: Open HTTP Port 80 in Firewall
         ufw:
           rule: allow
           port: "80"
           proto: tcp

       - name: Deploy HTML file
         copy:
           src: /path/to/local/index.html
           dest: /var/www/html/index.html
           mode: "0644"

       - name: Ensure Apache is Running
         service:
           name: apache2
           state: started
   ```

4. **Explanation of Playbook Tasks**:

   - **Install Apache**: Ensures the Apache2 web server is installed and the package list is updated.
   - **Start Apache**: Ensures the Apache service is running and enabled to start at boot.
   - **Open Firewall Ports**: Configures UFW to allow traffic on port 80 for HTTP.
   - **Deploy HTML**: Copies a static `index.html` file to the server's web root.
   - **Check Service Status**: Ensures Apache is still running after deployment.

5. **Run the Ansible Playbook**:

   - Execute the playbook using the `ansible-playbook` command:
     ```bash
     ansible-playbook -i hosts setup-web.yml
     ```

6. **Verify the Web Application**:
   - After the playbook finishes, you can verify that the web server is running by visiting `http://<server-ip>` in a browser.
   - The deployed `index.html` file should be served.

#### Example Output of Playbook Run:

```bash
PLAY [webservers] ******************************************************************

TASK [Gathering Facts] *************************************************************
ok: [192.168.1.10]

TASK [Install Apache Web Server] ***************************************************
changed: [192.168.1.10]

TASK [Start Apache Service] ********************************************************
changed: [192.168.1.10]

TASK [Open HTTP Port 80 in Firewall] ***********************************************
changed: [192.168.1.10]

TASK [Deploy HTML file] ************************************************************
changed: [192.168.1.10]

TASK [Ensure Apache is Running] ****************************************************
ok: [192.168.1.10]

PLAY RECAP *************************************************************************
192.168.1.10               : ok=6    changed=5    unreachable=0    failed=0
```

#### Result:

The Ansible playbook successfully sets up a basic web application infrastructure in a lab environment, including a web server, firewall configuration, and deployment of a sample HTML file.

---

### 7. **Detailed Walkthrough of Building a Simple Application Using Gradle in a Lab Setting**

#### Aim:

To build a simple Java application using Gradle in a lab environment, demonstrating the process of creating, building, and running a Java project.

#### Procedure:

1. **Install Gradle**:

   - Ensure Gradle is installed on your system. You can install Gradle using a package manager, such as `apt` (on Ubuntu) or `brew` (on macOS).
     ```bash
     sudo apt update
     sudo apt install gradle
     ```

2. **Create a Simple Java Project**:

   - Start by creating a new directory for your project:
     ```bash
     mkdir HelloGradle
     cd HelloGradle
     ```

3. **Initialize Gradle in the Project Directory**:

   - Use the following command to initialize a new Gradle project with the `java` plugin:
     ```bash
     gradle init --type java-application
     ```
   - This command will create the basic structure for a Java project:
     ```
     HelloGradle/
     ├── build.gradle
     ├── gradle/
     ├── gradlew
     ├── gradlew.bat
     ├── settings.gradle
     └── src/
         └── main/
             └── java/
                 └── App.java
         └── test/
             └── java/
                 └── AppTest.java
     ```

4. **Understanding the `build.gradle` File**:

   - The `build.gradle` file is automatically created and contains basic settings:

     ```groovy
     plugins {
         id 'application'
     }

     repositories {
         mavenCentral()
     }

     dependencies {
         testImplementation 'junit:junit:4.12'
     }

     application {
         mainClassName = 'App'
     }
     ```

   - This script:
     - Defines the project as a Java application.
     - Specifies that `junit` is used for testing.
     - Defines the main class to be `App`.

5. **Write a Simple Java Program**:

   - Open the `src/main/java/App.java` file and write a simple "Hello, World!" program:
     ```java
     public class App {
         public static void main(String[] args) {
             System.out.println("Hello, Gradle!");
         }
     }
     ```

6. **Build the Application**:

   - Use Gradle to compile the Java program:
     ```bash
     gradle build
     ```
   - Gradle will compile the source files, run tests (if any), and generate a JAR file in the `build/libs` directory.

7. **Run the Application**:

   - To run the built application, use:

     ```bash
     gradle run
     ```

   - The output should be:
     ```
     > Task :run
     Hello, Gradle!
     ```

8. **Customize the Build Script**:

   - You can add more tasks or configurations to the `build.gradle` file. For example, if you want to create a task to clean up the build directory:
     ```groovy
     task clean(type: Delete) {
         delete rootProject.buildDir
     }
     ```

9. **Test the Application**:
   - You can also run unit tests using JUnit. The `src/test/java/AppTest.java` file contains basic test setup. You can run tests using:
     ```bash
     gradle test
     ```

#### Result:

A simple Java application is successfully built, compiled, and run using Gradle. This demonstrates how to manage and automate Java builds using Gradle in a lab environment.

---

### 8. **Commands and Configurations Required to Install Ansible and Configure Ansible Roles for Playbook Writing**

#### Aim:

To install Ansible and configure Ansible roles for writing structured playbooks in a lab environment.

#### Procedure:

1. **Install Ansible**:

   - Ensure your system is updated and install Ansible.

     ```bash
     sudo apt update
     sudo apt install ansible -y
     ```

   - Verify that Ansible is installed correctly:
     ```bash
     ansible --version
     ```

2. **Configure an Inventory File**:

   - Create a file named `hosts` that will define the inventory of servers to manage:

     ```ini
     [webservers]
     192.168.1.10 ansible_user=ubuntu
     ```

   - This file groups the servers under the `webservers` group and specifies SSH access using the `ansible_user`.

3. **Write a Basic Playbook**:

   - Create a directory for the Ansible project:

     ```bash
     mkdir ansible_project
     cd ansible_project
     ```

   - Write a basic playbook file `playbook.yml`:

     ```yaml
     ---
     - hosts: webservers
       become: yes

       tasks:
         - name: Install Apache
           apt:
             name: apache2
             state: present
     ```

4. **Run the Playbook**:

   - Execute the playbook with the `ansible-playbook` command:
     ```bash
     ansible-playbook -i hosts playbook.yml
     ```

5. **Understanding Roles in Ansible**:

   - Roles allow you to break down the playbook into reusable components. To create a role, use the following command:

     ```bash
     ansible-galaxy init webserver
     ```

   - This creates a directory structure for the `webserver` role:
     ```
     webserver/
     ├── defaults/
     ├── files/
     ├── handlers/
     ├── meta/
     ├── tasks/
     ├── templates/
     ├── tests/
     └── vars/
     ```

6. **Structure a Role**:

   - In the `tasks/main.yml` file inside the `webserver` role, define the tasks:

     ```yaml
     ---
     - name: Install Apache
       apt:
         name: apache2
         state: present

     - name: Start Apache Service
       service:
         name: apache2
         state: started
     ```

7. **Use the Role in a Playbook**:

   - Modify your playbook to include the role:
     ```yaml
     ---
     - hosts: webservers
       become: yes
       roles:
         - webserver
     ```

8. **Run the Playbook with the Role**:

   - Execute the playbook, and Ansible will use the tasks defined in the `webserver` role:
     ```bash
     ansible-playbook -i hosts playbook.yml
     ```

9. **Testing the Role**:
   - Ansible automatically tests your role with the `tests` directory. Run the following to test your role:
     ```bash
     ansible-playbook tests/test.yml -i hosts
     ```

#### Result:

Ansible is successfully installed, and roles are configured for playbook writing. Roles are used to modularize tasks and make the playbook reusable.

---

### 9. **How Jenkins Facilitates Continuous Integration (CI) for Software Development Projects in a Lab Scenario**

#### Aim:

To demonstrate how Jenkins facilitates Continuous Integration (CI) by automating the building, testing, and integration of software code in a lab environment.

#### Procedure:

1. **Install Jenkins**:

   - Install Jenkins in the lab environment, using either a local machine or a cloud instance like Azure. If using Ubuntu, install Jenkins with the following commands:

     ```bash
     sudo apt update
     sudo apt install openjdk-11-jdk -y
     wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
     sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
     sudo apt update
     sudo apt install jenkins -y
     ```

   - Start Jenkins:

     ```bash
     sudo systemctl start jenkins
     ```

   - Access Jenkins via the web interface by opening `http://localhost:8080`.

2. **Set up a Jenkins Pipeline**:

   - Login to Jenkins and create a new job by clicking **New Item**, choosing **Pipeline**, and naming it appropriately (e.g., `Java CI Pipeline`).

3. **Configure the Jenkins Pipeline**:

   - In the pipeline configuration, select the option to define a pipeline using a script.
   - Use the following example `Jenkinsfile` for a basic CI pipeline:

     ```groovy
     pipeline {
         agent any

         stages {
             stage('Clone Repository') {
                 steps {
                     git 'https://github.com/user/repository.git'
                 }
             }
             stage('Build') {
                 steps {
                     sh './gradlew build'
                 }
             }
             stage('Test') {
                 steps {
                     sh './gradlew test'
                 }
             }
         }

         post {
             always {
                 archiveArtifacts artifacts: '**/build/libs/*.jar', allowEmptyArchive: true
                 junit 'build/test-results/test/*.xml'
             }
         }
     }
     ```

   - This pipeline:
     - **Clones** the repository.
     - **Builds** the project using Gradle.
     - **Runs tests** and generates reports.
     - **Archives artifacts** and test results.

4. **Triggering the Pipeline**:

   - You can configure Jenkins to automatically trigger builds using a webhook that listens for pushes or pull requests from GitHub.
   - Configure the GitHub repository to send push notifications to Jenkins using a webhook that hits Jenkins at: `http://<JENKINS_URL>/github-webhook/`.

5. **Build Automation**:

   - Jenkins continuously monitors the repository for changes. Each time a developer pushes a commit, Jenkins automatically triggers the pipeline to:
     - Pull the latest code.
     - Build the project.
     - Run tests.
     - Generate reports and notifications.

6. **Test Result Analysis**:

   - After the tests are run, Jenkins provides detailed reports on test results, allowing the development team to identify issues early in the development cycle.

7. **Notifications and Feedback**:
   - You can configure Jenkins to send email or Slack notifications when a build is successful or fails, providing instant feedback to developers.

#### Example:

- If your project is hosted on GitHub, Jenkins can be configured to pull code from the `main` branch, build it, run tests, and archive build artifacts like JAR files.
- Upon a successful build, Jenkins can notify the development team via Slack that the build has passed. If the build fails, Jenkins can notify the team, and the developers can quickly address any issues.

#### Result:

Jenkins automates the entire build and test process, continuously integrating new code into the project and ensuring that any issues are identified early. This enhances collaboration, speeds up development, and improves code quality.

---

### 10. **Writing and Utilizing Ansible Playbooks for Automating Infrastructure Tasks and Deployments in a Lab Exercise**

#### Aim:

To demonstrate how to write and utilize Ansible playbooks to automate infrastructure tasks and deployments in a lab setting.

#### Procedure:

1. **Install Ansible**:

   - Install Ansible in your lab environment if it’s not already installed:
     ```bash
     sudo apt update
     sudo apt install ansible -y
     ```

2. **Create an Inventory File**:

   - The inventory file defines the hosts or servers where you want to deploy the application or run tasks. Create an `inventory` file with the following content:
     ```ini
     [webservers]
     192.168.56.101 ansible_user=vagrant
     ```

3. **Write a Basic Ansible Playbook**:

   - Ansible playbooks contain tasks to be executed on remote servers. Create a file named `deploy_web.yml`:

     ```yaml
     ---
     - name: Deploy Web Application
       hosts: webservers
       become: yes
       tasks:
         - name: Install Apache
           apt:
             name: apache2
             state: present
             update_cache: yes

         - name: Start Apache
           service:
             name: apache2
             state: started

         - name: Deploy Application Code
           copy:
             src: /path/to/local/application
             dest: /var/www/html/

         - name: Ensure Apache is running
           service:
             name: apache2
             state: started
     ```

   - This playbook:
     - Installs Apache web server on the target machines.
     - Starts the Apache service.
     - Copies the web application code to `/var/www/html/`.
     - Ensures Apache is running after the deployment.

4. **Run the Playbook**:

   - Execute the playbook on the web server using the `ansible-playbook` command:

     ```bash
     ansible-playbook -i inventory deploy_web.yml
     ```

   - This command will connect to the server, execute the tasks, and deploy the web application.

5. **Role-Based Structure**:

   - To modularize the playbook, break it into roles. For example, to deploy a web server, create the following role structure:

     ```bash
     ansible-galaxy init webserver
     ```

   - Modify the `tasks/main.yml` in the `webserver` role:

     ```yaml
     ---
     - name: Install Apache
       apt:
         name: apache2
         state: present
         update_cache: yes

     - name: Start Apache
       service:
         name: apache2
         state: started
     ```

6. **Create a Playbook Utilizing the Role**:

   - Use the `webserver` role in the main playbook:
     ```yaml
     ---
     - hosts: webservers
       become: yes
       roles:
         - webserver
     ```

7. **Test Automation**:

   - You can also automate testing using Ansible by creating playbooks that verify the correctness of the deployment. For instance, you could use a `curl` command to check if the web application is running:

     ```yaml
     - name: Test Application
       command: curl http://localhost
       register: result

     - name: Check if Application is Running
       debug:
         msg: "Application is running: {{ result.stdout }}"
     ```

8. **Automate Multi-Stage Deployments**:
   - Ansible can also automate multi-stage deployments by defining multiple hosts in the inventory file and executing tasks in parallel across multiple environments (e.g., staging, production).

#### Example:

- Imagine deploying a Python web application. Ansible can be used to:
  - Install the Python runtime and necessary packages.
  - Set up Nginx as the reverse proxy.
  - Deploy the code and restart the service.
  - Validate that the application is up and running.

#### Result:

Ansible playbooks efficiently automate the deployment and management of infrastructure. Using roles, you can modularize tasks, making them reusable and easier to maintain across different environments. Playbooks allow the lab environment to be managed as code, automating routine tasks and ensuring consistent, reliable deployments.
