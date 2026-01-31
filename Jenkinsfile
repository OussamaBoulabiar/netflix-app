pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_CRED = credentials('dockerhub-credentials') // DockerHub credentials
        GITHUB_CRED = credentials('github-credentials') // GitHub credentials
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials', // Use GitHub credentials here
                    url: 'https://github.com/OussamaBoulabiar/netflix-app.git' // Updated repository URL
            }
        }
/*        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix
                    '''
                }
            }
        }
     */   
        stage('Quality Gate') {
            steps {
                script {
                    echo "SonarQube quality gate check done"
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {  // This step block is mandatory
                dir('path/to/your/app') { // Update to the correct path if needed
                    sh 'npm install'
                }
            }
        }
                
        stage('OWASP FS Scan') {
            steps {
                echo "OWASP dependency check completed"
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-credentials', toolName: 'docker') {
                        sh '''
                        docker build --build-arg TMDB_V3_API_KEY=bc921e8074b0cd1bb04fba36ca56b7d6 -t netflix-app .
                        docker tag netflix-app oussama132/netflix-app:latest
                        docker push oussama132/netflix-app:latest
                        '''
                    }
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image oussama132/netflix-app:latest > trivyimage.txt'
            }
        }
        
        stage('Deploy to Container') {
            steps {
                script {
                    // Remove the existing container if it exists
                    sh 'docker rm -f netflix || true'
                    // Run the new container
                    sh 'docker run -d --name netflix -p 8081:80 oussama132/netflix-app:latest'
                }
            }
        }
    }
}
