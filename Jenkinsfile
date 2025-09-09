pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "jithendarramagiri1998"
        APP_NAME = "hello-web"
        IMAGE_TAG = "latest"
        AWS_REGION = "ap-south-1"   // update region if needed
        CLUSTER_NAME = "hello-cluster"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Jithendarramagiri1998/hello-dear.git'
            }
        }

        stage('Build with Maven') {
            steps {
                dir('hello-web') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('hello-web') {
                    withSonarQubeEnv('sonarqube') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('hello-web') {
                    sh 'ls -l target/'  // check WAR file exists
                    sh "docker build -f Dockerfile -t $DOCKERHUB_USER/$APP_NAME:$IMAGE_TAG ."
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    dir('hello-web') {
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push $DOCKERHUB_USER/$APP_NAME:$IMAGE_TAG"
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                dir('hello-web') {
                    sh "aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME"
                    sh "kubectl apply -f hello-web-deployment.yaml"
                    sh "kubectl apply -f hello-web-service.yaml"
                }
            }
        }
    }
}
