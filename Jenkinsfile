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
