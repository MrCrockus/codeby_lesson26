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
                withSonarQubeEnv('sonar') {
                    script {
                        if (CHANGED_FILES.tokenize('\n').any { it.startsWith('helloworld/') }) {
                            sh 'mvn sonar:sonar -f helloworld/pom.xml -Dsonar.projectKey=l27_s -Dsonar.host.url=$SONAR_HOST_URL'
                        }
                        if (CHANGED_FILES.tokenize('\n').any { it.startsWith('hellojenkins/') }) {
                            sh 'mvn sonar:sonar -f hellojenkins/pom.xml -Dsonar.projectKey=l27_s -Dsonar.host.url=$SONAR_HOST_URL'
                        }
                        if (CHANGED_FILES.tokenize('\n').any { it.startsWith('hellodevops/') }) {
                            sh 'mvn sonar:sonar -f hellodevops/pom.xml -Dsonar.projectKey=l27_s -Dsonar.host.url=$SONAR_HOST_URL'
                        }
                    }
                }
            }
        }


        stage('Quality Gate') {
          steps {
              timeout(time: 10, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
    }
}
        stage('Build & Test') {
            parallel {
                stage('HelloWorld') {
                    when { expression { CHANGED_FILES.tokenize('\n').any { it.startsWith('helloworld/') } } }
                    steps {
                        dir('helloworld') {
                            sh 'mvn clean verify'
                        }
                    }
                }
                stage('HelloJenkins') {
                    when { expression { CHANGED_FILES.tokenize('\n').any { it.startsWith('hellojenkins/') } } }
                    steps {
                        dir('hellojenkins') {
                            sh 'mvn clean verify'
                        }
                    }
                }
                stage('HelloDevops') {
                    when { expression { CHANGED_FILES.tokenize('\n').any { it.startsWith('hellodevops/') } } }
                    steps {
                        dir('hellodevops') {
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
