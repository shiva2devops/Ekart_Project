pipeline {
    agent any
    
    tools{
        maven 'maven3'
        jdk 'jdk17'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/shiva2devops/Ekart_Project.git'
            }
        }
    

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
    

        stage('Unit Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
    

        stage('SonarQube Analysis ') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                    -Dsonar.java.binaries=. '''
                    
              }
            }
        }
 
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                        sh "mvn deploy -DskipTests=true"
                 }
            }
        }
        stage('Build Docker Image') {
            steps {
                script{
                    
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        sh "docker build -t mshiva7396/ekart:v1 ."
                     
                     }
                 }
            }
            }
        stage('Trivy Scan') {
            steps {
                sh "trivy image mshiva7396/ekart:v1 > trivy-report.txt"
                
            }
            }
        stage('Push Docker Image') {
            steps {
                script{
                    
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        sh "docker push mshiva7396/ekart:v1"
                     
                     }
                 }
            }
            }
        }
    }
