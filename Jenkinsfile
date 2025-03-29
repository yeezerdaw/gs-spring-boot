pipeline {
    agent any
    
    tools {
        maven 'Maven'
        jdk 'JDK17'  // Using JDK17
    }
    
    environment {
        DOCKER_REGISTRY = "yourusername"
        IMAGE_NAME = "spring-boot-sample"
        IMAGE_TAG = "${BUILD_NUMBER}"
        SONAR_TOKEN = "squ_4319aa99eecd3d6ac874e7f1b4c7f3a9af711854"
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Ensure you're pulling from the 'main' branch
                git branch: 'main', url: 'https://github.com/yeezerdaw/gs-spring-boot.git'
                echo 'Git repository cloned successfully'
            }
        }
        
        stage('Build') {
            steps {
                // Navigate to the complete directory and build with Maven
                dir('complete') {
                    sh 'mvn clean package -DskipTests -Djava.version=17'
                }
                echo 'Project built successfully'
            }
        }
        
        stage('Test') {
            steps {
                // Run tests in the complete directory
                dir('complete') {
                    sh 'mvn test'
                }
                echo 'Tests executed successfully'
            }
            post {
                always {
                    // Publish test results
                    junit 'complete/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                // Run SonarQube analysis in the complete directory
                dir('complete') {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                            mvn sonar:sonar \
                            -Dsonar.projectKey=spring-boot-sample \
                            -Dsonar.host.url=http://localhost:9000 \
                            -Dsonar.login=${SONAR_TOKEN}
                        '''
                    }
                }
                echo 'SonarQube analysis completed'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                // Create Docker image
                dir('complete') {
                    sh '''
                        docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .
                        docker tag ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest
                    '''
                }
                echo 'Docker image built successfully'
            }
        }
        
        stage('Push Docker Image') {
            steps {
                // Push Docker image to registry
                withCredentials([string(credentialsId: 'docker-hub-password', variable: 'DOCKER_HUB_PASSWORD')]) {
                    sh '''
                        echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_REGISTRY} --password-stdin
                        docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest
                    '''
                }
                echo 'Docker image pushed to registry'
            }
        }
        
        stage('Deploy Container Locally') {
            steps {
                // Run container locally
                sh '''
                    docker stop ${IMAGE_NAME} || true
                    docker rm ${IMAGE_NAME} || true
                    docker run -d -p 8080:8080 --name ${IMAGE_NAME} ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
                echo 'Container deployed locally on port 8080'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed!'
        }
        always {
            // Clean up workspace
            cleanWs()
        }
    }
}
