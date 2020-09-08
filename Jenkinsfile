pipeline {
    agent any
    environment {
        registry = 'marcosdejesus/ngix-demo'
    }

    stages {
        stage('Linting HTML and Dockerfile') {
            steps {
                sh 'tidy -q -e *.html'
            }
        }
        stage ("lint dockerfile") {
            agent {
                docker {
                    image 'hadolint/hadolint:latest-debian'
                }
            }
            steps {
                sh 'hadolint Dockerfile'
            }
        }
        stage('Build Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub-credentials', toolName: 'Stable docker') {
                        docker.build(registry).push(BUILD_NUMBER)
                    }
                }
            }
        }
        stage('Identify the environment') {
            environment {
                ACTIVE_ROLE = """${sh(
                    returnStdout: true,
                    script: 'kubectl get service nginx-service -o=jsonpath=\'{.spec.selector.role}{"\\n"}\''
                )}"""
            }
            steps {
                echo ACTIVE_ROLE
            }
        }
        stage('Clean up'){
            steps{
                sh "docker rmi $registry:${env.BUILD_NUMBER}"
            }
        }
    }
}
