@Library('Shared') _
pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "vishalldwivedi/todo-backend"
        FRONTEND_IMAGE = "vishalldwivedi/todo-frontend"
        GITOPS_REPO = "https://github.com/Vishalldwivedi/todo-gitops.git"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout App Repo') {
            script{
                    code_checkout("https://github.com/Vishalldwivedi/tooodoapp","main")
                }
        }

        stage("Trivy: Filesystem scan"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }

        stage("OWASP: Dependency check"){
            steps{
                script{
                    owasp_dependency()
                }
            }
        }
        
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("Sonar","wanderlust","wanderlust")
                }
            }
        }
        
        stage("SonarQube: Code Quality Gates"){
            steps{
                script{
                    sonarqube_code_quality()
                }
            }
        }
        
       
        
        stage('Build Docker Images') {
            steps {
               

                sh """
                docker build -t ${BACKEND_IMAGE}:${TAG} TodoSummaryAssistant/Backend/todo-summary-assistant
                docker build -t ${FRONTEND_IMAGE}:${TAG} TodoSummaryAssistant/Frontend/todo
                """
            }
        }

        stage('Docker Login (Secure)') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                }
            }
        }

        stage('Push Images') {
            steps {
                sh """
                docker push ${BACKEND_IMAGE}:${TAG}
                docker push ${FRONTEND_IMAGE}:${TAG}
                """
            }
        }

        stage('GitOps Update (Update Kubernetes Repo)') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {

                    sh """
                    rm -rf todo-gitops
                    git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/vishalldwivedi/todo-gitops.git

                    cd todo-gitops

                    sed -i "s|vishalldwivedi/todo-backend:.*|vishalldwivedi/todo-backend:${TAG}|" k8s/backend-deployment.yaml
                    sed -i "s|vishalldwivedi/todo-frontend:.*|vishalldwivedi/todo-frontend:${TAG}|" k8s/frontend-deployment.yaml

                    git add .
                    git commit -m "Update images to ${TAG}"
                    git push origin main
                    """
                }
        }
    }

    post {
        success {
            echo "CI + GitOps completed successfully!"
        }
        failure {
            echo "Pipeline failed — fix errors."
        }
    }
    }
}

