pipeline {
    agent any

    tools {
        // Define the tools to be used in the pipeline
        maven 'Maven 3.9.9' // Maven tool name as defined in Jenkins Global Tool Configuration
        jdk 'JDK 17' // JDK tool name as defined in Jenkins Global Tool Configuration
    }

    environment {
        // Hardcoded SonarQube token
        SONAR_TOKEN = 'squ_4319aa99eecd3d6ac874e7f1b4c7f3a9af711854'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Assuming Maven is used for build
                    sh 'mvn clean install'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Run tests
                    sh 'mvn test'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('myserver') {
                    script {
                        // Running SonarQube analysis
                        sh '''
                            mvn sonar:sonar \
                            -Dsonar.projectKey=spring-boot-sample \
                            -Dsonar.host.url=http://localhost:9000 \
                            -Dsonar.login=${SONAR_TOKEN}
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh 'docker build -t my-image .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push Docker image to registry
                    sh 'docker push my-image'
                }
            }
        }

        stage('Deploy Container Locally') {
            steps {
                script {
                    // Run Docker container locally
                    sh 'docker run -d -p 8080:8080 my-image'
                }
            }
        }
    }

    post {
        always {
            cleanWs()  // Clean workspace after build
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed.'
        }
    }
}
