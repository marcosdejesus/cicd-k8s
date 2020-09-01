pipeline {
    agent any


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
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Build Image') {
            steps {
                def builtImage = docker.build('marcosdejesus/nginx-demo')
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
