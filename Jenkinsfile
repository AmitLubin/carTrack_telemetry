def E2E = 'False'
def TAG = "1.0.0"
def JARAN = ""
def JARSIM = ""

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

        stage('Put-version'){
            when {
                branch 'release/*'
            }

            agent {
                docker {
                    image 'maven:3.6.3-jdk-8'
                    args '--network jenkins_jenkins_network'
                    // reuseNode true
                }
            }

            steps {
                script {
                    def version = env.BRANCH_NAME.split('/')[1]
                    def tag_c = 0
                    sshagent(credentials: ['GitlabSSHprivateKey']){
                        tag_c = sh(script: "git ls-remote --tags origin | grep ${version} | wc -l", returnStdout: true)
                    }
                    TAG = "${version}.${tag_c}"
                    sh "${MVN} versions:set -DnewVersion=${TAG}"
                    sh "${MVN} deploy"
                }
            }
        }

        stage('Maven-deploy'){
            when {
                anyOf {
                    branch 'main'
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
                script {
                    if (env.BRANCH_NAME == 'main') {
                        echo "Skipped!"
                        sh "${MVN} deploy -DskipTests"
                    } else {
                        echo "Tested!"
                        sh "${MVN} deploy"
                    }
                    sh "${MVN} deploy"
                }
                stash(name: 'jar', includes: 'target/*.jar')
            }
        }

        stage('Curl-artifactory-and-E2E'){
            when {
                anyOf {
                    branch 'main'
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

            // steps {
            //     script {
            //         def analytics = sh(script: "curl -u admin:Al12341234 -X GET 'http://artifactory:8082/artifactory/api/storage/libs-snapshot-local/com/lidar/analytics/99-SNAPSHOT/'", returnStdout: true)
            //         def simulator = sh(script: "curl -u admin:Al12341234 -X GET 'http://artifactory:8082/artifactory/api/storage/libs-snapshot-local/com/lidar/simulator/99-SNAPSHOT/'", returnStdout: true)

            //         def jsonSlurper = new groovy.json.JsonSlurper()
            //         def parsedAnalytics = jsonSlurper.parseText(analytics)
            //         def parsedSimulator = jsonSlurper.parseText(simulator)

            //         // Extract the JAR file URI
            //         def jarAnalytics = parsedAnalytics.children.find { it.uri.endsWith(".jar") }?.uri
            //         def jarSimulator = parsedSimulator.children.find { it.uri.endsWith(".jar") }?.uri

            //         echo "${jarAnalytics}"
            //         echo "${jarSimulator}"

            //         sh "curl -u admin:Al12341234 -O http://artifactory:8082/artifactory/libs-snapshot-local/com/lidar/analytics/99-SNAPSHOT/${jarAnalytics}"
            //         sh "curl -u admin:Al12341234 -O 'simulator.jar' 'http://artifactory:8082/artifactory/libs-snapshot-local/com/lidar/simulator/99-SNAPSHOT${jarSimulator}'"
            //         sh "ls"
            //         sh "java -cp simulator.jar:analytics.jar:target/telemetry-99-SNAPSHOT.jar com.lidar.simulation.Simulator"
            //     }
            // }
        }

        stage("Release-artifactory"){
            when {
                branch 'release/*'
            }

            agent {
                docker {
                    // image 'openjdk:8-jre-alpine3.9'
                    image 'maven:3.6.3-jdk-8'
                    args '--network jenkins_jenkins_network'
                }
            }

            steps {
                script {
                    def analytics = sh(script: "curl -u admin:Al12341234 -X GET 'http://artifactory:8082/artifactory/api/storage/libs-snapshot-local/com/lidar/analytics/'", returnStdout: true)
                    def simulator = sh(script: "curl -u admin:Al12341234 -X GET 'http://artifactory:8082/artifactory/api/storage/libs-snapshot-local/com/lidar/simulator/'", returnStdout: true)

                    def jsonSlurper = new groovy.json.JsonSlurper()
                    def parsedAnalytics = jsonSlurper.parseText(analytics)
                    def parsedSimulator = jsonSlurper.parseText(simulator)

                    // Extract the JAR file URI
                    def jarAnalytics = parsedAnalytics.children.find { it.uri.endsWith(".jar") }?.uri
                    def jarSimulator = parsedSimulator.children.find { it.uri.endsWith(".jar") }?.uri

                    echo "${jarAnalytics}"
                    echo "${jarSimulator}"

                    JARAN = jarAnalytics
                    JARSIM = jarSimulator
                }
            }
        }

        stage('Test'){
            when {
                branch 'release/*'
            }

            agent {
                docker {
                    // image 'openjdk:8-jre-alpine3.9'
                    image 'maven:3.6.3-jdk-8'
                    args '--network jenkins_jenkins_network'
                }
            }

            steps {
                sh "curl -u admin:Al12341234 -o analytics.jar 'http://artifactory:8082/artifactory/libs-snapshot-local/com/lidar/analytics${JARAN}'"
                sh "curl -u admin:Al12341234 -o simulator.jar 'http://artifactory:8082/artifactory/libs-snapshot-local/com/lidar/simulator${JARSIM}'"
                sh "ls -l"
                sh "java -cp simulator.jar:analytics.jar:target/telemetry-99-SNAPSHOT.jar com.lidar.simulation.Simulator"
            }
        }

        

        


    }

    post {
        always {
            cleanWs()
        }
    }
}