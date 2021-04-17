pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo '运行构建自动化'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'build/libs/spring-boot-0.0.1-SNAPSHOT.jar'
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                sshPublisher(
                    failOnError: true,
                    continueOnError: false,
                    publishers: [
                        sshPublisherDesc(
                            configName: 'staging',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'build/libs/spring-boot-0.0.1-SNAPSHOT.jar',
                                    removePrefix: 'build/libs/',
                                    remoteDirectory: '/tmp',
                                    execCommand: 'sudo systemctl stop SpringBootHelloWorld && sudo rm -rf /opt/springboot/spring-boot-0.0.1-SNAPSHOT.jar && sudo cp /tmp/spring-boot-0.0.1-SNAPSHOT.jar /opt/springboot && sudo systemctl start SpringBootHelloWorld'
                                )
                            ]
                        )
                    ]
                )
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Staging 环境的部署是否一切正常？'
                milestone(1)
                sshPublisher(
                    failOnError: true,
                    continueOnError: false,
                    publishers: [
                        sshPublisherDesc(
                            configName: 'production',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'build/libs/spring-boot-0.0.1-SNAPSHOT.jar',
                                    removePrefix: 'build/libs/',
                                    remoteDirectory: '/tmp',
                                    execCommand: 'sudo systemctl stop SpringBootHelloWorld && sudo rm -rf /opt/springboot/spring-boot-0.0.1-SNAPSHOT.jar && sudo cp /tmp/spring-boot-0.0.1-SNAPSHOT.jar /opt/springboot && sudo systemctl start SpringBootHelloWorld'
                                )
                            ]
                        )
                    ]
                )
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull k8sshuceshi/springboot-hello-world-docker:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop springboot-hello-world-docker\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm springboot-hello-world-docker\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name springboot-hello-world-docker -p 8080:8080 -d k8sshuceshi/springboot-hello-world-docker:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
