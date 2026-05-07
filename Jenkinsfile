
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

                if (branchName.contains('/')) {
                    imageTagPrefix = branchName.substring(
                        branchName.lastIndexOf('/') + 1
                    )
                } else {
                    imageTagPrefix = branchName
                }

                env.IMAGE_TAG_PREFIX = imageTagPrefix

                if (branchName == 'main' || branchName.startsWith('release/')) {
                    echo "Non-production artifact build"
                    env.IS_DEPLOY_NONPROD = "false"
                    env.IS_DEPLOY_PROD = "true"
                    env.IS_PACKAGE = "true"
                }
                else if (branchName.startsWith('develop')) {
                    echo "DEVELOPMENT artifact build"
                    env.IS_DEPLOY_NONPROD = "true"
                    env.IS_DEPLOY_PROD = "false"
                    env.IS_PACKAGE = "true"
                }
                else if (branchName.startsWith('feature/')) {
                    echo "FEATURE branch build (build only)"
                   env.IS_DEPLOY_NONPROD = "false"
                    env.IS_DEPLOY_PROD = "false"
                    env.IS_PACKAGE = "false"
                }

                echo "Image tag prefix = ${env.IMAGE_TAG_PREFIX}"
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
                script {
                    if (env.IS_PACKAGE == "true") {
                        echo "Packaging artifact"
                        sh "${tool 'MAVEN3'}/bin/mvn package"
                    } else {
                        echo "Skipping packaging"
                    }
                }
            }
        }
        stage('Stage - Deploy') {
            steps {
                script {
                    if (env.IS_DEPLOY_PROD == "true") {
                        echo "Deploying production artifact"
                    } else if (env.IS_DEPLOY_NONPROD == "true") {
                        echo "Deploying non-production artifact"
                    } else {
                        echo "Skipping deployment"
                    }
                }
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
