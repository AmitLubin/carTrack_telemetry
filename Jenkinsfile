pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 20, unit: 'MINUTES')
    }

    environment {
        MVN="mvn -s settings.xml"
    }

    stages {
        stage('Git-checkout'){
            steps {
                deleteDir()
                checkout scm
            }
        }

        stage('Maven-deploy'){
            agent {
                docker {
                    image 'maven:3.6.3-jdk-8'
                    args '--network jenkins_jenkins_network'
                }
            }

            steps {
                sh "${MVN} deploy"
                stash(name: 'jar', includes: 'target/*.jar')
                sh "ls"
                sh "ls target"
                sh "rm ? '*.jar' 99-SNAPSHOT"
            }
        }

        stage('Curl-artifactory'){
            agent {
                docker {
                    // image 'openjdk:8-jre-alpine3.9'
                    image 'maven:3.6.3-jdk-8'
                    args '--network jenkins_jenkins_network'
                }
            }

            steps {
                // unstash(name: 'jar')
                sh "curl -u admin:Al12341234 -O 'http://localhost:8082/artifactory/libs-snapshot-local/com/lidar/analytics/99-SNAPSHOT/analytics-99-20230911.074016-1.jar"
                sh "ls -l"
                sh "ls target"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}