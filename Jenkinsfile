pipeline {
    agent { label 'slave1' }
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKER_IMAGE = "mukulnvm/zomato-clone"
        SONAR_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/MukulNvm/Zomato_Clone_Food_Delivery_Web_Application.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SONAR_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=zomato-clone \
                        -Dsonar.projectName="Zomato Clone" \
                        -Dsonar.sources=. \
                        -Dsonar.language=py
                    '''
                }
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        
        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} -t ${DOCKER_IMAGE}:latest ."
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image-report.html ${DOCKER_IMAGE}:latest"
            }
        }
        
        stage('Docker Push') {
            steps {
                sh '''
                    echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }
        
        stage('Update Manifest') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    sh '''
                        git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/MukulNvm/zomato-manifest.git
                        cd zomato-manifest
                        sed -i "s|mukulnvm/zomato-clone:.*|mukulnvm/zomato-clone:${BUILD_NUMBER}|g" deployment.yaml
                        git config user.email "jenkins@ci.com"
                        git config user.name "Jenkins"
                        git add .
                        git commit -m "Update image to build ${BUILD_NUMBER}"
                        git push
                    '''
                }
            }
        }
    }
    
    post {
        always {
            sh 'docker logout || true'
        }
    }
}
