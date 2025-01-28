pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
        dockerTool 'docker'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/soundar3446/jen.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage('SonarQube') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('sonar-server') {
                        sh '''
                            $SCANNER_HOME/bin/sonar-scanner \
                            -Dsonar.projectName="shopping-cart" \
                            -Dsonar.java.binaries=./target/classes \
                            -Dsonar.projectKey="shopping-cart"
                        '''
                    }
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }

        stage('Docker Build') {
            steps {
                withDockerRegistry(credentialsId: 'docker-creds', url: 'https://index.docker.io/v1/') {
                    sh "docker build -t shopping-cart:latest -f docker/Dockerfile ."
                    sh "docker tag shopping-cart soundar4336/shopping-cart:latest"
                    sh "docker push soundar4336/shopping-cart:latest"
                }
            }
        }
    }

    post {
        success {
            emailext(
                to: 'rajaponpriya@gmail.com',
                subject: "Build #${env.BUILD_NUMBER} - Success",
                body: "Build #${env.BUILD_NUMBER} succeeded.\nCheck console output: ${env.BUILD_URL}"
            )
        }
        failure {
            emailext(
                to: 'rajaponpriya@gmail.com',
                subject: "Build #${env.BUILD_NUMBER} - Failed",
                body: "Build #${env.BUILD_NUMBER} failed.\nCheck console output: ${env.BUILD_URL}"
            )
        }
    }
}
