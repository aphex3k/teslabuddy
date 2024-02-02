pipeline {
    agent any
    environment {
        tag = sh(returnStdout: true, script: 'basename $(mktemp XXXXX) | tr "[:upper:]" "[:lower:]"').trim()
        AUTH = credentials('gitea_docker_deploy')
    }
    triggers {
        cron(BRANCH_NAME == "main" ? '''H H H * *''' : "")
    }
    stages {
        stage('Checkout') {
            steps {
                withCredentials([gitUsernamePassword(credentialsId: 'gitea-jenkins', gitToolName: 'Default')]) {
                    checkout scm
                    echo 'Clean'
                    sh 'git clean -xdf'
                }                
            }
        }
        stage('Dockerfile') {
            steps {
                script {                        
                    sh "docker build -t docker-teslabuddy-${tag} ."
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh 'docker login -u $AUTH_USR -p $AUTH_PSW gitea.codingmerc.com'
                    }                     

                    def push = env.BRANCH_NAME == 'main' ? "--push" : ""
                    sh "docker buildx create --name buildx${tag}"
                    sh "docker buildx build --builder buildx${tag} --progress=plain --platform=linux/amd64 --tag 'gitea.codingmerc.com/michael/teslabuddy:latest' ${push} --compress --squash ."
                }
            }
        }    
    }
    post {
        always {
            script {
                sh "docker rmi docker-teslabuddy-${tag} || echo 'no docker image generated...'"
                sh "docker buildx rm --force buildx${tag} || echo 'no buildx image generated...'"
                sh "docker buildx prune --all --force"
                sh "docker buildx rm --all-inactive --force"
            }
        }
    }
}
