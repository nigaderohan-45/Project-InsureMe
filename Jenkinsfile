pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        S3_BUCKET = "project-insure-me-build-artifacts-b31"
        REGION = "us-east-2"
        warFile = "target/Insurance-0.0.1-SNAPSHOT.jar"
        IMAGE_NAME = "abhipraydh96/insure-b31"
    }

    stages {

        stage('code-pull') {
            steps {
                git branch: 'main', url: 'https://github.com/abhipraydhoble/Project-InsureMe.git'
            }
        }

        stage('code-build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('code-push') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                credentialsId: 'aws-cred',
                secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh 'aws s3 cp ${warFile} s3://${S3_BUCKET}/Artifacts/ --region ${REGION}'
                }
            }
        }

        stage('docker-image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:v${BUILD_NUMBER} .'
            }
        }

        stage('image-push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-cred',
                    passwordVariable: 'dockerHubPassword',
                    usernameVariable: 'dockerHubUser')]) {

                    sh 'docker login -u ${dockerHubUser} -p ${dockerHubPassword}'
                    sh 'docker push ${IMAGE_NAME}:v${BUILD_NUMBER}'
                }
            }
        }

        stage('code-deploy') {
            steps {
                sh 'docker rm -f insure-me || true'
                sh 'docker run -d --name insure-me -p 8089:8089 ${IMAGE_NAME}:v${BUILD_NUMBER}'
            }
        }
    }
}
