pipeline{
    agent any
    tools{
        jdk 'jdk17'
        // nodejs 'node16'
        maven 'maven'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        // GIT_REPO_NAME = "Tetris-manifest"
        // GIT_USER_NAME = "rameshkumarvermagithub"      # change your Github Username here
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/rameshkumarvermagithub/complete-prodcution-e2e-pipeline.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=complete-prodcution-e2e-pipeline \
                    -Dsonar.projectKey=complete-prodcution-e2e-pipeline '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar' 
                }
            } 
        }
        // stage('Install Dependencies') {
        //     steps {
        //         sh "npm install"
        //     }
        // }
        stage('Maven Build'){
            steps{
                sh 'mvn clean package'
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
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){ 
                       // sh "docker pull chydinma/app:1"
                      sh "docker build -t rameshkumarverma/complete-prodcution-e2e-pipeline ."
                       // sh "docker tag chydinma/app:1 rameshkumarverma/myprojectapp:latest"
                       sh "docker push rameshkumarverma/complete-prodcution-e2e-pipeline:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image rameshkumarverma/complete-prodcution-e2e-pipeline:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to Kubernets'){
            steps{
                script{
                    // dir('Manifests') {
                      withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                      sh 'kubectl delete --all pods'
                      sh 'kubectl apply -f myapp-deployment.yml'
                      sh 'kubectl apply -f myapp-service.yml'
                      }   
                    }
                }
            }
        }
    }
}
