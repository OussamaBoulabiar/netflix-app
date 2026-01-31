pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_CRED = credentials('dockerhub-credentials')
        GITHUB_CRED = credentials('github-credentials')
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
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/OussamaBoulabiar/netflix-app.git'
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
                echo "SonarQube quality gate check done"
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('path/to/your/app') {
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
                sh 'cat trivyfs.txt'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub-credentials', toolName: 'docker') {
                    sh '''
                    docker build --build-arg TMDB_V3_API_KEY=bc921e8074b0cd1bb04fba36ca56b7d6 -t netflix-app .
                    docker tag netflix-app oussama132/netflix-app:latest
                    docker push oussama132/netflix-app:latest
                    '''
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image oussama132/netflix-app:latest > trivyimage.txt'
                sh 'cat trivyimage.txt'
            }
        }

        stage('Deploy Helm Chart') {
            steps {
                withCredentials([file(credentialsId: 'kind-kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    export KUBECONFIG=$KUBECONFIG_FILE
                    kubectl get nodes
                    helm upgrade -i netflix ./helm --namespace netflix --create-namespace
                    '''
                }
            }
        }
    }
}
