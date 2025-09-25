pipeline {
    agent any

    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }

    environment {
        SNAP_REPO       = 'vprofile-snapshot'
        NEXUS_USER      = 'admin'
        NEXUS_PASS      = 'Aakhil123@'
        RELEASE_REPO    = 'vprofile-release'
        CENTRAL_REPO    = 'vpro-maven-central'
        NEXUSIP         = '65.2.145.14'
        NEXUSPORT       = '8081'
        NEXUS_GRP_REPO  = 'vpro-maven-group'
        NEXUS_LOGIN     = 'nexuslogin'
        SONARSERVER     = 'sonarserver' // Must match Jenkins SonarQube server name
    }

    stages {
        stage('Build') {
            steps {
                sh '''
                    envsubst < settings.xml > settings_resolved.xml
                    mvn -s settings_resolved.xml -DskipTests install
                '''
            }
            post {
                success {
                    echo 'Build completed. Archiving WAR files...'
                    archiveArtifacts artifacts: '**/target/*.war', allowEmptyArchive: true
                }
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn -s settings_resolved.xml test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Integration Test') {
            steps {
                sh 'mvn -s settings_resolved.xml verify -DskipUnitTests'
            }
            post {
                always {
                    junit 'target/failsafe-reports/*.xml'
                }
            }
        }

        stage('Code Analysis with Checkstyle') {
            steps {
                sh 'mvn -s settings_resolved.xml checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Checkstyle analysis completed.'
                    archiveArtifacts artifacts: 'target/checkstyle-result.xml', allowEmptyArchive: true
                }
            }
        }

        stage('Code Analysis with SonarQube') {
            steps {
                script {
                    // Resolve SonarQube Scanner tool
                    def scannerHome = tool 'SONARSCANNER'
                    
                    withSonarQubeEnv(SONARSERVER) {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile-repo \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
        failure {
            echo "Build failed!"
        }
        success {
            echo "Build completed successfully!"
        }
    }
}
