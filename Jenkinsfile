pipeline {
    agent any
    tools{
        nodejs "nodejs"
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/17J/Three-Tier-Full-Stack.git'
            }
        }
        stage('Trivy Repo Scan') {
            steps {
                sh 'trivy repo --format table -o repo-report.html https://github.com/17J/Three-Tier-Full-Stack.git'
            } 
        }
        stage('Sonar Code Scan') {
            steps {
               withSonarQubeEnv('sonar') {
                    
                    sh ''' $SCANNER_HOME/bin/sonar-scanner  -Dsonar.projectName=threetierfull \
                           -Dsonar.sources=.\
                           -Dsonar.projectKey=threetierfull '''
   
                    }
            }
        }
        stage('Quality Gate') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
               dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
               dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Docker Deploy') {
            steps {
                sh 'docker-compose up -d'
            }
        }
    }
}
