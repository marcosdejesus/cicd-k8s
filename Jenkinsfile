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
                sh "docker rmi $registry:${env.BUILD_NUMBER}"
            }
        }
        stage('Identify the environment') {
            steps {
                script {
                    try {
                        env.ACTIVE_ROLE = sh(returnStdout: true, script: 'kubectl get service nginx-service -o=jsonpath=\'{.spec.selector.role}{"\\n"}\'').trim()
                        env.TARGET_ROLE = env.ACTIVE_ROLE == "blue" ? "green" : "blue"
                    } catch (err) {
                        env.TARGET_ROLE = 'blue'
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                dir('./k8config') {
                    sh "sed -e 's/%TARGET_ROLE%/${env.TARGET_ROLE}/g' -e 's/%IMAGE_VERSION%/${env.BUILD_NUMBER}/g' template/deploymentTemplate.yaml > Deployment.yaml"
                    sh "sed s/%TARGET_ROLE%/${env.TARGET_ROLE}/g template/test-serviceTemplate.yaml > test-endpoint.yaml"
                    sh 'kubectl apply -f Deployment.yaml -f test-endpoint.yaml'
                    sh 'kubectl get -f test-endpoint.yaml -o=jsonpath=\'{.status.loadBalancer.ingress[0].hostname}{"\\n"}\''
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    env.TEST_ENDPOINT = sh(returnStdout: true, script: 'kubectl get service nginx-test-service -o=jsonpath=\'{.status.loadBalancer.ingress[0].hostname}{"\\n"}\'').trim()
                }
                input message: "Is the deployment at $env.TEST_ENDPOINT working as desired?"
            }
        }
        stage('Switch production') {
            input {
                message "Do you want to switch all the traffic to the new deployment?"
            }
            steps {
                echo 'Switching'
                dir('./k8config') {
                    sh "sed s/%TARGET_ROLE%/${env.TARGET_ROLE}/g template/serviceTemplate.yaml > endpoint.yaml"
                    sh 'kubectl apply -f endpoint.yaml'
                }
            }
        }
    }
}
