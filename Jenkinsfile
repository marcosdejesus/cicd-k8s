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
            steps {
                script {
                    env.ACTIVE_ROLE = sh(returnStdout: true, script: 'kubectl get service nginx-service -o=jsonpath=\'{.spec.selector.role}{"\\n"}\'').trim()
                    env.TARGET_ROLE = env.ACTIVE_ROLE == "blue" ? "green" : "blue"
                }
                sh 'printenv'
                //echo ACTIVE_ROLE
                //echo TARGET_ROLE
            }
        }
        stage('Deploy') {
            steps {
                dir('./k8config') {
                    sh "sed -e 's/%TARGET_ROLE%/${env.TARGET_ROLE}/g' -e 's/%IMAGE_VERSION%/${env.BUILD_NUMBER}/g' template/deploymentTemplate.yaml > Deployment.yaml"
                    sh "sed s/%TARGET_ROLE%/${env.TARGET_ROLE}/g template/test-serviceTemplate.yaml > test-endpoint.yaml"
                    sh 'kubectl apply -f Deployment.yaml test-endpoint.yaml'
                    sh 'kubectl get -f test-endpoint.yaml -o=jsonpath=\'{.status.loadBalancer.ingress[0].hostname}{"\\n"}\''
                }
            }
        }
        stage('Clean up'){
            steps{
                sh "docker rmi $registry:${env.BUILD_NUMBER}"
            }
        }
    }
}
