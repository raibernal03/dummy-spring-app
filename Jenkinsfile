pipeline {
    agent any
    tools {
        maven 'MAVEN3'
    }

    stages {
        stage('Step - SCM') {
            steps {
                git branch: 'main',
                        url: 'https://github.com/raibernal03/dummy-spring-app.git'
            }
        }
        stage('Step - Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Branch Detection') {
            steps {
                script {
                    def branch = env.BRANCH_NAME

                    if (branch == 'develop') {
                        echo "Dev environment build"
                    } else if (branch.startsWith('feature/')) {
                        echo "Feature branch build only"
                    } else if (branch.startsWith('release/')) {
                        echo "Staging candidate build"
                    } else if (branch == 'main') {
                        echo "Production build"
                    }
                }
            }
        }

    }
}
