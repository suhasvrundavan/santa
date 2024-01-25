pipeline {
    agent{label 'slave-1'}
    
    tools{
        maven 'maven3'
        jdk 'jdk17'
    }
    
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Tests') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=santa \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=santa '''
                }
            }
        }
        
        
        stage('Build Application') {
            steps {
                sh "mvn package"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker-1' ) {
                        sh "docker build -t santa:latest ."
                    }
                }
            }
        }
        
        stage('Tag & Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker-1' ) {
                        sh "docker tag santa:latest suhasjv9/santa:latest"
                        sh "docker push suhasjv9/santa:latest"
                    }
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker-1' ) {
                        sh "docker run -d -p 8081:8080 suhasjv9/santa:latest"
                    }
                }
            }
        }
        
    }
    
    post {
        always{
            emailext(
                subject: "Pipeline Status: ${BUILD_NUMBER}",
                body: ''' <html>
                            <body>
                                <p> Build Status: ${BUILD_STATUS}</p>
                                <p> Build Number: ${BUILD_NUMBER}</p>
                                <p> Check the <a href="${BUILD_URL}"> console output</a>.</p>
                            </body>
                          </html> ''',
                to: 'xz86627@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html'
                )
        }
    }
}
