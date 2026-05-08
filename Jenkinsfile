
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
                    def imageTagPrefix = ''

                    if (branchName.contains('/')) {
                        imageTagPrefix = branchName.substring(
                        branchName.lastIndexOf('/') + 1
                    )}
                    else {
                        imageTagPrefix = branchName
                    }

                    env.IMAGE_TAG_PREFIX = imageTagPrefix

                    if (branchName == 'main' || branchName.startsWith('release/')) {
                        echo 'Non-production artifact build'
                        env.IS_DEPLOY_NONPROD = 'false'
                        env.IS_DEPLOY_PROD = 'true'
                        env.IS_PACKAGE = 'true'
                    }
                    else if (branchName.startsWith('develop')) {
                            echo 'DEVELOPMENT artifact build'
                            env.IS_DEPLOY_NONPROD = 'true'
                            env.IS_DEPLOY_PROD = 'false'
                            env.IS_PACKAGE = 'true'
                    }
                    else if (branchName.startsWith('feature/')) {
                            echo 'FEATURE branch build (build only)'
                            env.IS_DEPLOY_NONPROD = 'false'
                            env.IS_DEPLOY_PROD = 'false'
                            env.IS_PACKAGE = 'false'
                    }

                        echo "Image tag prefix = ${env.IMAGE_TAG_PREFIX}"
                }
            }
        }
        stage('Stage - Build') {
            steps {
                sh "${tool 'MAVEN3'}/bin/mvn clean compile "
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
                    def appName = 'dummy-spring-app'
                    def buildNumber = env.BUILD_NUMBER
                    def branchName = env.BRANCH_NAME.trim()
                    def artifactName = ''

                    // Maven build (correct lifecycle)
                    sh "${tool 'MAVEN3'}/bin/mvn clean package"

                    // Find actual jar produced
                    def jarFile = sh(
                        script: 'ls target/*.jar | grep -v original',
                        returnStdout: true
                    ).trim()

                    if (branchName.startsWith('feature/')) {
                        def feature = branchName.tokenize('/').last()
                        artifactName = "${appName}-${feature}-${buildNumber}.jar"
                    }
                    else if (branchName == 'develop') {
                        artifactName = "${appName}-develop-${buildNumber}-SNAPSHOT.jar"
                    }
                    else if (branchName.startsWith('release/')) {
                        def version = branchName.tokenize('/').last()
                        artifactName = "${appName}-${version}-RC${buildNumber}.jar"
                    }
                    else {
                        artifactName = "${appName}-${buildNumber}.jar"
                    }

                    env.ARTIFACT_NAME = artifactName

                    echo "Renaming ${jarFile} → ${artifactName}"
                    sh "cp ${jarFile} target/${artifactName}"
                }
            }
        }
        stage('Check Docker') {
            steps {
                sh '''
            echo $PATH
            which docker || echo "NO DOCKER"
            docker version || echo "NO ACCESS"
        '''
            }
        }
        stage('Stage - Docker Build & Push') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME.trim()

                    if (branchName.startsWith('feature/')) {
                        sh "echo 'Skipping Docker build for feature branch'"
                    }
                    else if (branchName == 'develop' || branchName.startsWith('release/') || branchName == 'main') {
                        sh 'docker version'
                    }
                }
            }
        }

        stage('Stage - Deploy') {
            steps {
                script {
                    if (env.IS_DEPLOY_PROD == 'true') {
                        echo 'Deploying production artifact'
                    } else if (env.IS_DEPLOY_NONPROD == 'true') {
                        echo 'Deploying non-production artifact'
                    } else {
                        echo 'Skipping deployment'
                    }
                }
            }
        }

        stage('Stage - Branch Info') {
            steps {
                echo "Current branch is: ${env.BRANCH_NAME}"
            }
        }
        stage('Stage - Archive Artifact') {
            steps {
                script {
                    if (env.IS_PACKAGE == 'true') {
                        echo 'Archiving artifact'
                        archiveArtifacts artifacts: "target/${env.ARTIFACT_NAME}", fingerprint: true
                    } else {
                        echo 'No artifact to archive'
                    }
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    echo 'Building Docker image...'

                    def imageTag = env.ARTIFACT_NAME.replace('.jar', '')

                    echo "Docker tag will be: ${imageTag}"

                // next step: docker build command goes here
                }
            }
        }
    }
}
// feature/* → "Build only"
// develop → "Dev artifact"
// main → "Production artifact"
