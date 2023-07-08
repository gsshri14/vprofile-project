pipeline {

    agent any

    tools {
        maven 'MAVEN3'
        jdk 'JDK11'
    }

    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = '963900'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '172.31.90.195'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }

    stages {

        stage('BUILD') {

            steps {
                sh 'mvn -s settings.xml -DskipTests clean install'
            }

            post {
                success {
                    echo 'Now archiving...'
                    archiveArtifacts artifacts: '**/*.war', fingerprint: true
                }
            }
        }

        stage('TEST') {

            steps {
                sh 'mvn -s settings.xml clean test'
            }
        }


        stage('SONAR ANALYSIS') {

            environment {
                scannerHome = tool 'sonarscanner'
            }
            steps {
                withSonarQubeEnv('sonarserver'){
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                                       -Dsonar.projectName=vprofile-repo \
                                       -Dsonar.projectVersion=1.0 \
                                       -Dsonar.sources=src/ \
                                       -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                                       -Dsonar.junit.reportsPath=target/surefire-reports/ \
                                       -Dsonar.jacoco.reportsPath=target/jacoco.exec '''
                }
            }
        }

        stage('QUALITY GATE') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('UPLOAD ARTIFACT TO NEXUS') {

            steps {
                    def warFilePath = "${env.WORKSPACE}/target/vprofile-v2.war"
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                        groupId: 'QA',
                        version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                        repository: "${RELEASE_REPO}",
                        credentialsId: "${NEXUS_LOGIN}",
                        artifacts: [
                            [artifactId: 'vproapp',
                             classifier: '',
                             file: warFilePath,
                             type: 'war']
                        ]
                     )
            }
        }

    }

}