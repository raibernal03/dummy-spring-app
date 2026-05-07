
pipeline {
    agent any

    tools {
        maven 'MAVEN3'
    }

    stages {
        stage('Stage - Branch Logic') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME.trim()
                    def imageTagPrefix = ""

                    // Generate feature image tag prefix
                    if (branchName.contains('/')) {
                        imageTagPrefix = branchName.substring(
                            branchName.lastIndexOf('/') + 1,
                            branchName.length()
                        )
                    } else {
                        imageTagPrefix = branchName
                    }

                    // Decide artifact type (VISIBLE OUTPUT NOW)
                    if (env.BRANCH_NAME == 'main' || branchName.startsWith('release/')) {
                        echo "PRODUCTION artifact build"
                    }
                    else if (branchName.startsWith('develop')) {
                        echo "DEVELOPMENT artifact build"
                    }
                    else if (branchName.startsWith('feature/')) {
                        echo "FEATURE branch build (build only)"
                    }

                    echo "Image tag prefix = ${imageTagPrefix}"
                }
            }
        }

        stage('Stage - Build') {
            steps {
                sh "${tool 'MAVEN3'}/bin/mvn clean compile"
            }
        }

        stage('Stage - Test') {
            steps {
                sh "${tool 'MAVEN3'}/bin/mvn test"
            }
        }

        stage('Stage - Package') {
            steps {
                sh "${tool 'MAVEN3'}/bin/mvn package"
            }
        }

        stage('Stage - Branch Info') {
            steps {
                echo "Current branch is: ${env.BRANCH_NAME}"
            }
        }
    }
}

// feature/* → "Build only"
// develop → "Dev artifact"
// main → "Production artifact"
