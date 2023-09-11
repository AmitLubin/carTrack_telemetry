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
                    def lastCommitMessage = sh(script: 'git log -1 --pretty=%B', returnStdout: true)
                    if (lastCommitMessage.contains("#e2e")) {
                        E2E = 'True'
                    } else {
                        sh "${MVN} package"
                    }
                }
            }
        }

        stage('Maven-deploy'){
            when {
                anyOf {
                    branch 'main'
                    branch 'release/*'
                    expression {
                        return (env.BRANCH_NAME =~ /^feature\/.*/ && E2E == 'True')
                    }
                }
            }

            agent {
                docker {
                    image 'maven:3.6.3-jdk-8'
                    args '--network jenkins_jenkins_network'
                }
            }

            steps {
                if (env.BRANCH_NAME == 'main') {
                    sh "${MVN} deploy -DskipTests"
                } else {
                    sh "${MVN} deploy"
                }
                sh "${MVN} deploy"
                // stash(name: 'jar', includes: 'target/*.jar')
            }
        }

        stage('Curl-artifactory'){
            when {
                anyOf {
                    branch 'main'
                    branch 'release/*'
                    expression {
                        return (env.BRANCH_NAME =~ /^feature\/.*/ && E2E == 'True')
                    }
                }
            }

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
                sh "curl -u admin:Al12341234 -O 'http://artifactory:8082/artifactory/libs-snapshot-local/com/lidar/simulator/99-SNAPSHOT/simulator-99-20230911.100821-1.jar'"
                sh "ls -l"
                sh "java -cp simulator-99-20230911.100821-1.jar:analytics-99-20230911.074016-1.jar:target/telemetry-99-SNAPSHOT.jar com.lidar.simulation.Simulator"
            }
        }


    }

    post {
        always {
            cleanWs()
        }
    }
}