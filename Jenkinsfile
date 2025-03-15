pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME = tools 'sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
               git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Prajwal-HE/Boardgame.git'
            }
        }
        
        stage('compile') {
            steps {
               sh "mvn compile"
            }
        }
        
        stage('test') {
            steps {
                sh "mvn test -DskipTests = true"
            }
        }
        
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('sonarqube analysis') {
            steps {
               withSonarQubeEnv('sonar-scanner') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Boardgame -Dsnoar.projectKey=Boardgame \
                         -Dsonar.java.binaries=.'''
                }
            }
        }
    
        stage('quality gate') {
            steps {
                 script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
        
        stage('Build stage') {
            steps {
              sh "mvn package"
            }
        }
        stage('publish to Nexus') {
            steps {
            withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy"
            }
            }
        }
        stage('Build and tag docker image') {
            steps {
             script{
                withDockerRegistry(credentialsId: 'Dock-cred', toolName: 'docker') {
                    sh "docker build -t dock4379/boardgame:latest ."
                }
            }
        }
        
         stage('Docker image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html dock4379/boardgame:latest"
            }
        }
        
        stage('Push docker image') {
            steps {
             script{
                withDockerRegistry(credentialsId: 'Dock-cred', toolName: 'docker') {
                    sh "docker push dock4379/boardgame:latest"
                }
            }
        }
        
        stage('Deploy to k8s') {
            steps {
              withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: ' https://172.31.18.163:6443') {
                    sh "kubectl apply -f deployment-service.yaml"
                    }
            }
        }
        stage('Deploy to k8s') {
            steps {
              withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: ' https://172.31.18.163:6443') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
            
    }
}
