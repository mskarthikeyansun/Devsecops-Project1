pipeline{
    agent any
    tools{
        jdk 'JDK8'
        nodejs 'NodeJS'
    }
    environment {
        SCANNER_HOME=tool 'sonnar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/ravipramoth/Devsecops-Project1.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
                    -Dsonar.projectKey=Game '''
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
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){   
                       sh "docker build -t devsecops_ad ."
                       sh "docker tag devsecops_ad promo286/devsecops_ad:latest "
                       sh "docker push promo286/devsecops_ad:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image promo286/devsecops_ad:latest > trivy.txt" 
            }
        }
stage('Deploy to container'){
            steps{
                sh 'docker run -d --name 2048 -p 3000:3000 promo286/devsecops_ad:latest'
            }
        }
stage('Deploy to kubernets'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                       sh 'kubectl apply -f deployment.yaml --validate=false
l'
                  }
                }
            }
        }

    }
}
