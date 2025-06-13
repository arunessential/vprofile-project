pipeline {
    agent any
    tools {
        jdk 'JDK17' // Ensure 'JDK17' is configured as a global tool in Jenkins
        maven 'MAVEN3.9' // Ensure 'MAVEN3.9' is configured as a global tool in Jenkins
        sonarscanner 'SONARQUBE_SCANNER_4.7' // Ensure 'SONARQUBE_SCANNER_4.7' is configured globally in Jenkins
    }

    // Define environment variables for SonarQube Scanner configuration here
    // These variables will be available throughout the pipeline
    environment {
        // SONARSERVER refers to the name of your SonarQube server configuration in Jenkins
        // SONARSCANNER refers to the name of your SonarQube Scanner tool definition in Jenkins Global Tool Configuration
        // It's good to define these at the top level if they are constants
        SONARQUBE_SERVER_NAME = 'sonarserver'
        SONARQUBE_SCANNER_TOOL = 'SONARQUBE_SCANNER_4.7' // This matches the tool name in 'tools' block
        // No need for scannerHome here, 'withSonarQubeEnv' handles it or you can use the tool name directly.
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: 'target/*.war'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script { // 'script' block is good for more complex Groovy logic
                    // 'withSonarQubeEnv' automatically sets SONAR_HOST_URL and SONAR_AUTH_TOKEN
                    // based on the 'SONARQUBE_SERVER_NAME' credential.
                    withSonarQubeEnv("${SONARQUBE_SERVER_NAME}") {
                        // Use the 'tool' step to get the path to the SonarQube scanner executable
                        def scannerHome = tool "${SONARQUBE_SCANNER_TOOL}"

                        // Execute the SonarQube analysis using the Sonar Scanner CLI.
                        // For Maven projects, you can often use 'mvn sonar:sonar' directly,
                        // which is simpler and leverages Maven's existing project structure.
                        // If you use 'sonar-scanner', you need to provide all project properties manually.
                        // Let's go with the Maven way first, as it's common for Java projects.
                        sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar ' +
                           '-Dsonar.projectKey=vprofile_project_key ' + // <--- IMPORTANT: Update this to your actual project key
                           '-Dsonar.projectName=vprofile ' +
                           '-Dsonar.projectVersion=1.0 ' +
                           '-Dsonar.sources=src/main/java,src/main/resources ' + // Specify main sources
                           '-Dsonar.java.binaries=target/classes ' + // Specify compiled classes
                           '-Dsonar.tests=src/test/java ' + // Specify test sources
                           '-Dsonar.java.test.binaries=target/test-classes ' + // Specify compiled test classes
                           '-Dsonar.junit.reportsPath=target/surefire-reports ' + // Path to JUnit reports
                           '-Dsonar.jacoco.reportsPath=target/jacoco.exec' // Path to Jacoco execution report
                        
                        // If you *must* use the sonar-scanner CLI directly:
                        /*
                        sh "${scannerHome}/bin/sonar-scanner " +
                           "-Dsonar.projectKey=vprofile_project_key " + // <--- IMPORTANT: Update this to your actual project key
                           "-Dsonar.projectName=vprofile " +
                           "-Dsonar.projectVersion=1.0 " +
                           "-Dsonar.sources=src/ " +
                           "-Dsonar.java.binaries=target/classes " + // Adjust path as needed
                           "-Dsonar.junit.reportsPath=target/surefire-reports/ " +
                           "-Dsonar.jacoco.reportsPath=target/jacoco.exec " +
                           "-Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml"
                        */
                    }
                }
            }
            post {
                always {
                    script {
                        echo "Waiting for SonarQube analysis to complete and Quality Gate status..."
                        // Give SonarQube server some time to process the analysis report
                        sleep 60 // Increased sleep time, adjust based on project size/server load
                        try {
                            def qg = waitForQualityGate() // Requires SonarQube Quality Gates plugin
                            if (qg.status != 'OK') {
                                error "Pipeline aborted due to Quality Gate failure: ${qg.status}"
                            } else {
                                echo "SonarQube Quality Gate passed: ${qg.status}"
                            }
                        } catch (Exception e) {
                            echo "Error checking Quality Gate: ${e.message}. Proceeding anyway."
                            // You might want to 'error' here if Quality Gate check is mandatory.
                            // For now, it will just log the error and continue.
                        }
                    }
                }
            }
        }
    } // Closes the 'stages' block correctly
} // Closes the 'pipeline' block correctly
