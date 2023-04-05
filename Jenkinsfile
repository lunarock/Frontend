def imageName="lunarock/frontend"
def dockerRegistry=""
def registryCredentials="dockerhub"
def dockerTag=""

pipeline {

    agent {
      label 'agent'
    }

    environment {
      scannerHome = tool 'SonarQube'
    }

    stages {
        stage('Clone repo Frontend') {
            steps {
		checkout scm
            }
        }
  
        stage('Install requirements') {
            steps {
                sh 'pip3 install -r requirements.txt'
            }
        }

        stage('Run tests') {
            steps {
                sh 'python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'
            }
        }

        stage('Analyse with Sonarqube') {
            steps {
              withSonarQubeEnv('SonarQube') {
                sh "${scannerHome}/bin/sonar-scanner"
              }
              timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
        }

        stage('Build app img') {
            steps {
                script {
                    dockerTag = "RC-${env.BUILD_ID}" // .${env.GIT_COMMIT.take(7)}
                    applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }
 
        stage ('Push img') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        }
    }

    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }

}
