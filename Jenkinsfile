pipeline {
    agent any

    tools {
        maven 'MAVEN3'
    }

    stages {

        stage('Build') {
            steps {
                sh "${tool 'MAVEN3'}/bin/mvn clean compile"
            }
        }

        stage('Test') {
            steps {
                sh "${tool 'MAVEN3'}/bin/mvn test"
            }
        }

        stage('Package') {
            steps {
                sh "${tool 'MAVEN3'}/bin/mvn package"
            }
        }

        stage('Branch Info') {
            steps {
                echo "Current branch is: ${env.BRANCH_NAME}"
            }
        }
    }
}