pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git changelog: false, poll: false, url: 'https://github.com/harishasapu/UBER_CLONE.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('SonarQube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Uber \
                    -Dsonar.projectKey=Uber'''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
            stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build &amp; Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){   
                       sh "docker build -t uber ."
                       sh "docker tag uber harishasapu/uber:latest "
                       sh "docker push harishasapu/uber:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh " trivy image --format table harishasapu/uber:latest " 

            }
        }
        stage("Docker Deploy"){
            steps{
                sh "docker run -d --name uber -p 3000:3000 harishasapu/uber:latest" 
            }
        }
        stage('Deploy to kubernets'){
            steps{
                script{
                    dir('K8S'){
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s-cred', namespace: '', restrictKubeConfigAccess: false, serverUrl: ''){
                             sh 'kubectl apply -f deployment.yml'
                             sh 'kubectl apply -f service.yml'
                        }
                    }
                }
            }
        }
    }
}
