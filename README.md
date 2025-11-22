JENKINES-CI-PROJECT
-----------------------------------------------------------------

vprofile-app Continuous Integration Pipeline
The vprofile-app is a Java-based application compatible with Java 17 and 21. This repository contains the source code and the Jenkins pipeline configuration used to automate the continuous integration (CI) lifecycle.

This documentation focuses solely on the automated CI process managed by Jenkins, detailing how code commits are automatically triggered, built, tested, analyzed for quality, and published as deployable artifacts.

CI Pipeline Overview
The Continuous Integration pipeline is orchestrated by Jenkins and designed to ensure code quality and build integrity with every change. The pipeline performs the following key actions:

Automated compilation and building of the Java application with Maven.

Execution of unit tests.

Static code analysis using Checkstyle.

Comprehensive code quality and security scanning via SonarQube.

Enforcement of quality gates before proceeding.

Publishing the final versioned artifact (WAR file) to a Sonatype Nexus repository.

Sending real-time build status notifications to a Slack channel.

Category,Tool,Description
Cloud Provider,AWS EC2,"Hosts the Jenkins, SonarQube, and Nexus servers."
CI Orchestration,Jenkins,Manages the entire automated pipeline flow.
Build Tool,Apache Maven 3.9,"Handles project building, dependency management, and testing."
Language Runtime,Java 17,Jenkins uses JDK 17 for build tasks.
Code Quality,SonarQube,"Performs static analysis to detect bugs, vulnerabilities, and code smells."
Artifact Repo,Sonatype Nexus,Stores dependencies and hosts the final deployable build artifacts.
Notifications,Slack,Provides instant alerts on pipeline success or failure.

Pipeline Trigger & Automation
This pipeline adheres to Continuous Integration principles by automating builds on every significant code change.

Trigger Mechanism: Source Code Management (SCM) Webhook.

Trigger Events: A push or commit event to the repository.

Target Branch: The pipeline is configured to trigger automatically only on changes made to the main branch.

Whenever code is merged or pushed to the main branch, a webhook notifies Jenkins, which immediately initiates the pipeline execution process defined below.

Pipeline Architecture Diagram
The following diagram illustrates the flow of execution defined in the Jenkinsfile:
graph TD
    A[Developer Push to 'main'] -->|Webhook Trigger| B(Jenkins Pipeline Start);
    B --> C{Stage: Build};
    C -->|mvn install| D[Compile & Package WAR];
    D --> E{Stage: Test};
    E -->|mvn test| F[Run Unit Tests];
    F --> G{Stage: Checkstyle};
    G -->|mvn checkstyle| H[Static Code Analysis];
    H --> I{Stage: SonarQube Analysis};
    I -->|sonar-scanner| J[Push Report to SonarServer];
    J --> K{Stage: Quality Gate};
    K -->|Wait for Webhook| L(SonarQube Pass/Fail Decision);
    L -- Pass --> M{Stage: Deploy to Nexus};
    L -- Fail --> Q[Fail Pipeline];
    M -->|Upload WAR| N[(Nexus Release Repo)];
    N --> O[Post-Action: Notify];
    Q --> O;
    O -->|Slack Plugin| P[Send Slack Notification];

   Detailed Pipeline Stages
The Jenkinsfile defines a declarative pipeline with the following distinct stages:

1. Build
Tool: Maven

Command: mvn -s settings.xml -DskipTests install

Description: Compiles the source code, downloads necessary dependencies from the Nexus group repository, and packages the application into a .war file. It skips tests in this stage to speed up the build.

Post-Action: On success, the generated WAR file (**/target/*.war) is archived as a Jenkins artifact for later use.

2. Test
Tool: Maven

Command: mvn -s settings.xml test

Description: Runs the unit tests defined in the project to ensure basic code functionality.

3. Checkstyle
Tool: Maven Checkstyle Plugin

Command: mvn -s settings.xml checkstyle:checkstyle

Description: Performs a static analysis of the code against predefined coding standards and generates a report.

4. SonarQube Analysis
Tool: SonarScanner

Description: The pipeline connects to the defined SonarQube server environment. The sonar-scanner tool is executed to analyze source code, test coverage reports (JaCoCo), and checkstyle reports. The results are published to the SonarQube dashboard under the project key vprofile.

5. Quality Gate
Description: This stage pauses the pipeline for up to 1 minute, waiting for a callback webhook from the SonarQube server. The pipeline will only proceed if the analysis meets the defined "Quality Gate" criteria (e.g., passing score on bugs, vulnerabilities, coverage). If the gate fails, the pipeline is aborted.

6. Deploy to Nexus
Tool: Nexus Artifact Uploader Plugin

Description: Upon successfully passing all previous stages and the quality gate, the final vprofile-v2.war artifact is versioned with the build ID and timestamp and uploaded to the vprofile-release repository on the Nexus server.

Notifications
The pipeline is configured to send post-build notifications to the #jenkins Slack channel.

Success: A green notification with the build number and a link to the Jenkins build page.

Failure: A red notification alerting the team to the failed build.

Environment Configuration
Security and configuration are managed using Jenkins Credentials and Environment variables, ensuring sensitive data is not hardcoded in the Jenkinsfile.

NEXUS_LOGIN: Jenkins credential ID for authenticating with the Nexus repository for uploads.

SONARSERVER / SONARSCANNER: Jenkins tool and environment configurations for connecting to the SonarQube instance.

Server IPs & Ports: Managed as environment variables (NEXUSIP, NEXUSPORT) for flexibility across environments. 
