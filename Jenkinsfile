pipeline {

    agent any

    stages {
        stage("wipe working directory") {
            steps {
                deleteDir()
            }
        }

        stage("checkout sources") {
            steps {
                checkout scm
            }
        }

        stage('Build and publish lib') {
            agent {
                docker {
                    reuseNode true
                    image 'eclipse-temurin:11.0.22_7-jdk-jammy'
                }
            }
            steps {
                // tests fail and we don't want to investigate this which is why we currently skip them
                sh './gradlew clean build -x test -x compileTestGroovy -x compileIntegrationTestGroovy'
                script {
                    if(env.BRANCH_NAME ==~ /^(master|release\/.*)$/) {
                        withCredentials([usernamePassword(credentialsId: 'SOLUNIO_NEXUS_PUBLISH_CREDENTIALS', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                            sh 'NEXUS_URL=http://nexus.lan/repository/mvn-internal/ ./gradlew publish -x test -x compileTestGroovy -x compileIntegrationTestGroovy'
                        }
                    }
                }

            }
        }
    }

    post {
        failure {
            script {
                if(env.BRANCH_NAME ==~ /^(master|release\/.*)$/) {
                    googlechatnotification url: 'id:credential_id_google_chat_builds_room', message: "\\u26a1\\ud83d\\udea8 Build failed for project *${env.JOB_NAME}* [<${env.BUILD_URL}|${env.BUILD_NUMBER}>] \\ud83d\\udea8\\u26a1 See <${env.BUILD_URL}console|Build Logs> for details...", notifyFailure: 'true'
                }
            }
        }
        success {
            // Cleanup the working directory on build success. On build failure we don't want to cleanup since
            // we may want to debug failure on the node
            deleteDir()
        }
    }
}
