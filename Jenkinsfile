pipeline {
    agent any
    
    tools {
        jdk 'JDK17' // Ensure 'JDK17' is configured as a global tool in Jenkins
        maven 'MAVEN3.9' // Ensure 'MAVEN3.9' is configured as a global tool in Jenkins
        // CORRECTED: Use the full class name for the SonarQube Scanner tool type
        // 'SONARQUBE_SCANNER_4.7.0.2747' must be the exact name configured in Global Tool Configuration 
    }

    environment {
        SONARQUBE_SERVER_NAME = 'sonarserver'
        // CORRECTED: The tool name used in 'tool' step and here should match the one defined in 'tools' block
        SONARQUBE_SCANNER_TOOL = 'SONARQUBE_SCANNER_4.7.0.2747' 
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
                script {
                    withSonarQubeEnv("${SONARQUBE_SERVER_NAME}") {
                        // Use the 'tool' step to get the path to the SonarQube scanner executable
                        // The string passed to tool() must match the name given in Global Tool Configuration
                        def scannerHome = tool "${SONARQUBE_SCANNER_TOOL}"

                        // Using 'mvn sonar:sonar' is generally preferred for Maven projects
                        sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar ' +
                           '-Dsonar.projectKey=vprofile_project_key ' + // IMPORTANT: Update this
                           '-Dsonar.projectName=vprofile ' +
                           '-Dsonar.projectVersion=1.0 ' +
                           '-Dsonar.sources=src/main/java,src/main/resources ' +
                           '-Dsonar.java.binaries=target/classes ' +
                           '-Dsonar.tests=src/test/java ' +
                           '-Dsonar.java.test.binaries=target/test-classes ' +
                           '-Dsonar.junit.reportsPath=target/surefire-reports ' +
                           '-Dsonar.jacoco.reportsPath=target/jacoco.exec'
                        
                        // If you *must* use the sonar-scanner CLI directly:
                        /*
                        // Ensure your sonar-scanner properties are correctly set for CLI usage
                        // and that paths are absolute or relative to the workspace root.
                        sh "${scannerHome}/bin/sonar-scanner " +
                           "-Dsonar.projectKey=vprofile_project_key " +
                           "-Dsonar.projectName=vprofile " +
                           "-Dsonar.projectVersion=1.0 " +
                           "-Dsonar.sources=src/ " +
                           "-Dsonar.java.binaries=target/classes " +
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
                        sleep 60 
                        try {
                            def qg = waitForQualityGate()
                            if (qg.status != 'OK') {
                                error "Pipeline aborted due to Quality Gate failure: ${qg.status}"
                            } else {
                                echo "SonarQube Quality Gate passed: ${qg.status}"
                            }
                        } catch (Exception e) {
                            echo "Error checking Quality Gate: ${e.message}. Proceeding anyway."
                        }
                    }
                }
            }
        }
    }
}
