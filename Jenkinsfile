pipeline {
    agent any

    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/anjurusatheesh/Shopping-cart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.host.url=http://localhost:9000 \
                    -Dsonar.login=squ_08676530db039c59edf3254ee19425f1481ca44d \
                    -Dsonar.projectKey=shopping-cart \
                    -Dsonar.projectName=shopping-cart \
                    -Dsonar.java.binaries=target/classes \
                    -X
                '''
            }
        }
        
        stage('Build Application') {
            steps {
                sh "mvn install -DskipTests=true"
            }
        }
        
        stage('Build & Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'anjurusatheesh-docker', toolName: 'Docker') {
                        sh "docker build -t anjurusatheesh/shopping:latest -f docker/Dockerfile ."
                        sh "docker push anjurusatheesh/shopping:latest"
                    }
                }
            }
        }
        
        stage('Docker Deploy To Local Container') {
            steps {
                sh '''
                    docker stop shopping-cart || true
                    docker rm shopping-cart || true
                    docker run -d --name shopping-cart -p 8070:8070 anjurusatheesh/shopping:latest
                '''
            }
        }
        
        stage('Executing Shell Script On Server') {
            steps {
                script {
                    sshagent(['EC2']) {
                        sh '''
                           ssh -t -t ubuntu@3.110.88.250 -o StrictHostKeyChecking=no << EOF
                            docker stop shopping-cart || true
                            docker rm shopping-cart || true
                            docker run -d --name shopping-cart -p 8070:8070 anjurusatheesh/shopping:latest
                            logout
                            EOF
                        '''
                    }
                }
            }
        }
    }
}
