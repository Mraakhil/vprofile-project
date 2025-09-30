pipeline {
    agent any
    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }

    environment {
        SNAP_REPO = 'vprofile-snapshot'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '65.2.145.14'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'Aakhil123@'

        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'SONARSCANNER'

        // Missing Nexus vars for artifact upload
        NEXUS_VERSION = 'nexus3'
        NEXUS_PROTOCOL = 'http'
        NEXUS_URL = '65.2.145.14:8081'
        NEXUS_REPOSITORY = 'vprofile-release'
        NEXUS_CREDENTIAL_ID = 'nexuslogin'
        ARTVERSION = '1.0.0'
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
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war', allowEmptyArchive: true
                }
            }
        }

        stage('UNIT TEST') {
            steps {
                sh 'mvn -s settings_resolved.xml test'
            }
        }

        stage('INTEGRATION TEST') {
            steps {
                sh 'mvn -s settings_resolved.xml verify -DskipUnitTests'
            }
        }

        stage('CODE ANALYSIS WITH CHECKSTYLE') {
            steps {
                sh 'mvn -s settings_resolved.xml checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
            steps {
                script {
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
        stage ("upload "){
            steps{
          nexusArtifactUploader(
          nexusVersion: 'nexus3',
          protocol: 'http',
          nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
          groupId: 'QA',
          version: "${env.BUID_ID}-${env.BUILD_TIMESTAP}",
          repository: "${RELEASE_REPO}",
          credentialsId: "${NEXUS_LOGIN}",
          artifacts: [
              [artifactId: projectName,
               classifier: '',
               file: 'my-service-' + version + '.jar',
               type: 'war']
            ]
          )
       }
    }
}
}
    
