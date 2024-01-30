pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=sangram-project \
                    -Dsonar.projectKey=sangram-project'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
             steps {
                 sh "trivy fs . > trivyfs.txt"
             }
         }
         stage("Docker Build & Push"){
             steps{
                 script{
                   withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){   
                      sh "docker build -t sangram/sangram-project:${BUILD_NUMBER} ."
                      sh "docker push sangram/sangram-project:${BUILD_NUMBER}"
                    }
                }
            }
        }
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image sangram/sangram-project:${BUILD_NUMBER} > trivyimage.txt" 
            }
        }
        stage('Deploy to Kubernets'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'f87a34a8-0e09-45e7-b9cf-6dc68feac670', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                        cat deploy.yaml
                        sed -i '' "s/32/${BUILD_NUMBER}/g" deploy.yaml
                        cat deploy.yaml
                        git add deploy.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push https://github.com/Sangramshinde97/sangramrepo-project/Kubernetes HEAD:master
                        '''                        
                    }
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
            to: 'sangramshinde97@gmail.com',                              
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
      }
}
