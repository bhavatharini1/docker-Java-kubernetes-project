pipeline {
    agent any

    environment {
        PATH = "/opt/homebrew/bin:$PATH:/usr/local/bin" // Include Docker in the PATH
        DOCKER_REGISTRY_CREDENTIALS = credentials('bhavatharinione') // Replace with your Docker credentials ID
        DOCKER_IMAGE_NAME = "prodoneee" // Use straight double quotes
        DOCKER_IMAGE_TAG = "latest" // You can customize the image tag
        KUBE_PATH = "/usr/local/bin" // Add the path to kubectl
    }

    stages {
        stage('Maven Build') {
            steps {
                dir('productcatalogue') {
                    sh 'mvn clean install'
                }
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    dir('productcatalogue') {
                        // Build a Docker image containing the Maven-built JAR
                        def dockerImage = docker.build("${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}", "-f Dockerfile .")

                        // Authenticate with Docker registry using credentials
                        withCredentials([usernamePassword(credentialsId: 'bhavatharinione', usernameVariable: 'DOCKERHUB_CREDENTIALS_USR', passwordVariable: 'DOCKERHUB_CREDENTIALS_PSW')]) {
                            sh "docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW"

                            // Push the Docker image to the registry
                            sh "docker push bhavatharinione/prodoneee:latest"
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                
                    script {
                        // Add /usr/local/bin to the PATH for kubectl
                        def updatedPath = "$KUBE_PATH:$PATH"
                        sh "export PATH=$updatedPath"
                        
                        // Now you can use kubectl with the updated PATH
                        sh "kubectl apply -f deployment.yaml"
                    }
                
            }
        }

        // Add more stages as needed
    }

    post {
        success {
            echo 'Maven build, Docker image push, and Kubernetes deployment succeeded!'
            // Add post-build actions
        }

        failure {
            echo 'Maven build, Docker image push, or Kubernetes deployment failed!'
            // Add failure handling or notifications
        }
    }
}
