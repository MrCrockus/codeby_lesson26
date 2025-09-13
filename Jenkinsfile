pipeline {
    agent any
    tools {
        maven 'M3'
    }
    environment {
        CHANGED_FILES = ""
    }

    stages {
        stage('Detect Changes') {
            steps {
                script {
                    def previousCommit = env.GIT_PREVIOUS_SUCCESSFUL_COMMIT ?: "HEAD~1"
                    CHANGED_FILES = sh(script: "git diff --name-only ${previousCommit} ${env.GIT_COMMIT} || true", returnStdout: true).trim()
                    echo "Changed files raw:\n${CHANGED_FILES}"
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    script {
                        if (CHANGED_FILES.tokenize('\n').any { it.startsWith('helloWorld/') }) {
                            sh 'mvn sonar:sonar -f helloWorld/pom.xml -Dsonar.projectKey=HelloWorld -Dsonar.host.url=$SONAR_HOST_URL'
                        }
                        if (CHANGED_FILES.tokenize('\n').any { it.startsWith('helloJenkins/') }) {
                            sh 'mvn sonar:sonar -f helloJenkins/pom.xml -Dsonar.projectKey=HelloJenkins -Dsonar.host.url=$SONAR_HOST_URL'
                        }
                        if (CHANGED_FILES.tokenize('\n').any { it.startsWith('helloDevops/') }) {
                            sh 'mvn sonar:sonar -f helloDevops/pom.xml -Dsonar.projectKey=HelloDevops -Dsonar.host.url=$SONAR_HOST_URL'
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
          steps {
            script {
              sleep 5
              timeout(time: 10, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
    }
}
        stage('Build & Test') {
            parallel {
                stage('HelloWorld') {
                    when { expression { CHANGED_FILES.tokenize('\n').any { it.startsWith('helloWorld/') } } }
                    steps {
                        dir('helloWorld') {
                            sh 'mvn clean verify'
                        }
                    }
                }
                stage('HelloJenkins') {
                    when { expression { CHANGED_FILES.tokenize('\n').any { it.startsWith('helloJenkins/') } } }
                    steps {
                        dir('helloJenkins') {
                            sh 'mvn clean verify'
                        }
                    }
                }
                stage('HelloDevops') {
                    when { expression { CHANGED_FILES.tokenize('\n').any { it.startsWith('helloDevops/') } } }
                    steps {
                        dir('helloDevops') {
                            sh 'mvn clean verify'
                        }
                    }
                }
            }
        }

        

        stage('Deploy') {
            steps {
                echo "Deploy only changed apps"
            }
        }
    }
}
