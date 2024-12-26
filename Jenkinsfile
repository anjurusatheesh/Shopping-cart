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
                        -Dsonar.login=squ_f3f9468591a4040066fff80059898fd7a60460b4 \
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
        
        stage('Build & push Docker Image') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'anjurusatheesh-docker', toolName: 'Docker') {
                            sh "docker build -t shopping:latest -f docker/Dockerfile ."
                            sh "docker tag shopping:latest anjurusatheesh/shopping:latest"
                            sh "docker push anjurusatheesh/shopping:latest"
                   }
               }
            }
        }
        stage('Docker Deploy To container') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'anjurusatheesh-docker', toolName: 'Docker') {
                             sh "docker run -d --name shopping-cart -p 8070:8070 anjurusatheesh/shopping:latest"
                    }       
                }
            }
        }
    }
}
