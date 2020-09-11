pipeline {
    agent any
    environment {
        registry = 'marcosdejesus/ngix-demo'
        ACTIVE_ROLE = ''
        TARGET_ROLE = ''
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
            steps {
                script {
                    env.ACTIVE_ROLE = sh returnStdout: true, script: 'kubectl get service nginx-service -o=jsonpath=\'{.spec.selector.role}{"\\n"}\''
                    //env.TARGET_ROLE = "${env.ACTIVE_ROLE.trim() == "blue" ? "green" : "blue"}"
                }
                sh 'printenv'
                //echo ACTIVE_ROLE
                //echo TARGET_ROLE
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying to $TARGET_ROLE"
            }
        }
        stage('Clean up'){
            steps{
                sh "docker rmi $registry:${env.BUILD_NUMBER}"
            }
        }
    }
}
