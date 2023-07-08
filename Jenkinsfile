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
    }

    stages {

        stage('BUILD') {

            steps {
                sh 'mvn -s settings.xml -DskipTests clean install'
            }

            post {
                success {
                    echo 'Now archiving...'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('TEST') {

            steps {
                sh 'mvn clean test'
            }
        }

        stage('CHECKSTYLE ANALYSIS') {

            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
        
    }

}