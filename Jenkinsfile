pipeline {
    agent { label 'slave1' }
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKER_IMAGE = "mukulnvm/zomato-clone"
        SONAR_HOME = tool 'sonar-scanner'
    }
    
    stages {
        // ... all your existing stages ...
        
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
