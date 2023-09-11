def E2E = 'False'

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

        stage('Check-last-commit'){
            when {
                branch 'feature/*'
            }

            agent {
                docker {
                    image 'maven:3.6.3-jdk-8'
                    args '--network jenkins_jenkins_network'
                }
            }

            steps {
                script {
                    echo "Entered!"
                    def lastCommitMessage = sh(script: 'git log -1 --pretty=%B', returnStdout: true)
                    echo "${lastCommitMessage}"
                    if (lastCommitMessage.contains("#e2e")) {
                        echo "E2E"
                        E2E = 'True'
                    } else {
                        sh "${MVN} package"
                        echo "Packaged"
                    }
                    
                }
            }
        }

        stage('Maven-deploy'){
            when {
                anyOf {
                    branch 'main'
                    branch 'release/*'

                }
            }

            agent {
                docker {
                    image 'maven:3.6.3-jdk-8'
                    args '--network jenkins_jenkins_network'
                }
            }

            steps {
                sh "${MVN} deploy"
                stash(name: 'jar', includes: 'target/*.jar')
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
                sh "curl -u admin:Al12341234 -O 'http://artifactory:8082/artifactory/libs-snapshot-local/com/lidar/analytics/99-SNAPSHOT/analytics-99-20230911.074016-1.jar'"

            }
        }


    }

    post {
        always {
            cleanWs()
        }
    }
}