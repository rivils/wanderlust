pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
    }
    stages {
        stage("Clone Code from GitHub") {
            steps {
                git url: "https://github.com/krishnaacharyaa/wanderlust.git", branch: "devops"
            }
        }

        stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("Sonar") {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                            $SONAR_HOME/bin/sonar-scanner \
                            -Dsonar.projectName=wanderlust \
                            -Dsonar.projectKey=wanderlust \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://localhost:9000 \
                            -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage("Sonar Quality Gate Scan") {
            steps {
                timeout(time: 2, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
    }
}
