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
            }
        }

        stage('Curl-artifactory'){
            steps {
                unstash(name: 'jar')
                sh "ls"
            }
        }
    }
}