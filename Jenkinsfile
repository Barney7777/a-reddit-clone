pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        APP_NAME = "reddit"
        RELEASE = "1.0.0"
        DOCKER_USER = "barneywang"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        GIT_REPO_NAME = "a-reddit-clone-gitops"
        GIT_USER_NAME = "Barney7777" 
    }
    stages {
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Barney7777/a-reddit-clone.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=reddit \
                    -Dsonar.projectKey=reddit '''
                }
            }
        }
        stage("quality gate"){
          steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.json"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t ${APP_NAME} ."
                       sh "docker tag ${APP_NAME} ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG} "
                       sh "docker push ${IMAGE_NAME}:${IMAGE_TAG} "
                    }
                }
            }
        }
        stage("TRIVY IMAGE SCAN"){
            steps{
                sh "trivy image ${IMAGE_NAME}:${IMAGE_TAG} > trivyimage.json"
            }
        }
        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${APP_NAME}:latest"                    
                }
            }
        }
        stage('Checkout Code From Gitops') {
            steps {
                git branch: 'main', url: 'https://github.com/Barney7777/a-reddit-clone-gitops.git'
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh """
                    cat deployment.yaml
                    sed -i 's|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' deployment.yaml
                    cat deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to GitHub") {
            steps {
                sh """
                    git config --global user.name "Barney7777"
                    git config --global user.email "wangyaxu7@gmail.com"
                    git add deployment.yaml
                    git commit -m "Updated Deployment Manifest"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh "git push https://github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} main"
                }
            }
         }
    }
    post {
        always {
           emailext attachLog: true,
               subject: "'${currentBuild.result}'",
               body: "Project: ${env.JOB_NAME}<br/>" +
                   "Build Number: ${env.BUILD_NUMBER}<br/>" +
                   "URL: ${env.BUILD_URL}<br/>",
               to: 'wangyaxu7@gmail.com',                              
               attachmentsPattern: 'trivyfs.json,trivyimage.json,dependency-check-report.xml'
        }
    }
}