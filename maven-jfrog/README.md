**✅ Maven + JFrog + Azure DevOps Lab**

**Build and Manage Spring Boot Apps on Azure VM with Maven, JFrog Artifactory, and Git**

**📌 Overview**

This guide walks you through:

*   Building a Spring Boot app using Maven.
*   Pushing artifacts to JFrog Artifactory.
*   Version control with Azure DevOps Git.
*   Managing versions dynamically with Maven plugins.
*   **Goal**: Build, manage, and deploy a Spring Boot application using Maven and JFrog Artifactory.
*   **Tools**: Maven, JFrog Artifactory, Azure DevOps, Git, OpenJDK 17.
*   **Key** **Steps**: Set up an Azure VM, build the application, push code to Azure DevOps, integrate with JFrog, and manage artifact versions.

**🔧 Prerequisites**

**1\. Azure VM Setup**

*   **Instance type**: Standard\_B2s or higher
*   **OS**: Ubuntu 22.04 LTS
*   **Disk Size**: 30 GB



**2\. Tools Installed on VM:**

| Bash |
| --- |
| sudo apt update && sudo apt install -y openjdk-17-jdk maven git jq net-tools |

**3\. Accounts:**

*   Azure DevOps (with SSH auth configured)
*   JFrog Artifactory (14-day trial or licensed version)
*   (Optional) Route 53 / Azure DNS for public access

**📂 Project Setup (Spring Boot)**

**1\. Clone and Build Spring Petclinic**

| bash |
| --- |
| git clone https://github.com/spring-projects/spring-petclinic.gitcd spring-petclinicmvn clean package |

**💻 Push to Azure DevOps Repo**

**1\. Initialize Git and Connect to Azure DevOps**

|  |
| --- |
| rm -rf .gitgit initgit add .git commit -m "Initial Commit"git remote add origin <your-azure-ssh-url>git push -u origin master |

**2\. Setup SSH Authentication (if not done)**

| bash |
| --- |
| ssh-keygen -t rsa -b 4096cat ~/.ssh/id_rsa.pub |

Add the public key in **Azure DevOps > User Settings > SSH Public Keys**

**🔁 Maven Lifecycle Reference**

| Phase | Command | Description |
| --- | --- | --- |
| Validate | mvn validate | Validate pom.xml structure |
| Compile | mvn compile | Compile .java into .class files |
| Package | mvn package | Build .jar or .war artifacts |
| Run | java -jar target/*.jar | Run the built Spring Boot app |
| Clean | mvn clean | Remove previous build artifacts |

**📦 JFrog Artifactory Integration**

**1\. Install JFrog Artifactory on Azure VM**

| bash |
| --- |
| sudo -icd /usr/local/bin/wget -O jfrog-deb-installer.tar.gz "https://releases.jfrog.io/artifactory/jfrog-prox/org/artifactory/pro/deb/jfrog-platform-trial-prox/7.98.9/jfrog-platform-trial-prox-7.98.9-deb.tar.gz"tar -xvzf jfrog-deb-installer.tar.gzcd jfrog-platform-trial-pro*apt install -y jq net-tools./install.sh |

**2\. Start the Artifactory service:**

| bash |
| --- |
| systemctl start artifactory.servicesystemctl start xray.servicesystemctl status artifactory.service |

**2\. Access JFrog**

Go to http://<public-ip>:8082 and set up your admin user and trial license.

**3\. Apply for a Trial License:**

Log into JFrog and apply for the 14-day trial license.

**4\. Configure Maven Repository**:

In JFrog Artifactory, go to **Artifacts** > **Set Me Up** > **Maven**.

Select the libs-release repository.

Enter your JFrog password to generate a token and settings (Don’t download latest version of JFrog, getting incorrect password issues at this step).

**⚙️ Maven Configuration with JFrog**

**1\. Create or edit /root/.m2/settings.xml and add the generated settings.**

| xml |
| --- |
| <settings><servers><server><id>central</id><username>admin</username><password>your-generated-token</password></server></servers><profiles><profile><id>jfrog</id><repositories><repository><id>central</id><url>http://jfrog.<your-domain>:8082/artifactory/libs-release</url><snapshots><enabled>true</enabled></snapshots></repository></repositories></profile></profiles></settings> |

Update the highlighted ones.

**Clarifying the Role of settings.xml**

The settings.xml file is a Maven configuration file that defines settings like repository authentication, proxy settings, and profiles. Maven looks for this file in two places:

*   **Global Location**: ~/.m2/settings.xml (e.g., C:\\Users\\<YourUsername>\\.m2\\settings.xml on Windows, /root/.m2/settings.xml on the VM).
*   **Project-Specific Location**: If you specify a custom settings.xml file in the project directory (e.g., spring-petclinic/settings.xml), you can tell Maven to use it with the -s flag (e.g., mvn -s settings.xml clean install).

**Default Behavior**

*   If you don’t specify a custom settings.xml with the -s flag, Maven will look for ~/.m2/settings.xml.
*   The settings.xml in ~/.m2/ is shared across all Maven projects on that machine, making it the standard place to configure global settings like JFrog Artifactory credentials.

**Do You Need to Create settings.xml in Your Local PC’s PetClinic Directory?**

Your instructor asked you to create settings.xml in your local PC’s PetClinic directory (spring-petclinic/settings.xml) and add it to .gitignore. Let’s evaluate this:

**Is It Necessary?**

*   **Not strictly necessary**, but it can be useful depending on your workflow.
*   By default, Maven will use ~/.m2/settings.xml on your local PC for all projects, including PetClinic. If you’ve already configured ~/.m2/settings.xml with the JFrog Artifactory settings (as shown in your previous workflow), you don’t _need_ a project-specific settings.xml in the PetClinic directory.
*   However, your instructor might have a specific reason for this request, such as:
    *   **Isolation**: A project-specific settings.xml ensures that the PetClinic project uses its own Maven settings, separate from other projects on your PC.
    *   **Team Consistency**: If your team is sharing the project, a project-specific settings.xml can be a template (though it should not be committed to Git due to sensitive data like passwords).

**2\. Modify pom.xml**:

*   Comment out (shortcut – Ctrl+/) any <execution> blocks (search for no-http-checkstyle-validation) under <build> in pom.xml to avoid conflicts. (It’s comes after dependencies – around line 220).

**Pushing Artifacts to JFrog**

**1\. Add Distribution Management to pom.xml - add the following to your pom.xml to specify where to deploy artifacts:**

| xml |
| --- |
| <distributionManagement><repository><id>central</id><name>libs-release</name><url>http://jfrog.<your-domain>:8082/artifactory/libs-release-local</url></repository><snapshotRepository><id>snapshots</id><name>libs-snapshot</name><url>http://jfrog.<your-domain>:8082/artifactory/libs-snapshot-local</url></snapshotRepository></distributionManagement> |

**🚀 Deploy Artifacts to JFrog**

**1\. Clean, Install & Deploy**

| mvn clean install deploy |
| --- |

**2\. Verify Artifacts**

Log in to JFrog → Artifacts → Check under libs-release-local

**🔁 Version Management with Maven**

**1\. Update Artifact Version Dynamically**

| Bash |
| --- |
| mvn versions:set -DnewVersion=1.0.1mvn clean install deploy |

Repeat for:

| Bash |
| --- |
| mvn versions:set -DnewVersion=2.0.0mvn clean install deploy |

**🧩 Maven Plugins for Versioning**

Add the following in pom.xml under <build><plugins>:

| xml |
| --- |
| <plugin><groupId>org.codehaus.mojo</groupId><artifactId>build-helper-maven-plugin</artifactId><version>3.2.0</version></plugin><plugin><groupId>org.codehaus.mojo</groupId><artifactId>versions-maven-plugin</artifactId><version>2.8.1</version></plugin> |

✅ Summary

This lab demonstrates how to:

*   Use Azure Building a **Spring Boot** application with **Maven**.
*   Managing dependencies via **pom.xml**.
*   Storing artifacts in **JFrog Artifactory**.
*   Versioning artifacts **incrementally**. Version and manage builds with **Maven plugins**.
*   Deploying to a private repository for reuse.
*   Integrate with **Azure DevOps Git** using SSH.
