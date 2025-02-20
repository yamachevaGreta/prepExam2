pipeline {
    agent any

    environment {
        DOTNET_VERSION = '6.0.x' // Define .NET version
    }

    stages {
        stage('Checkout Repository') {
            steps {
                script {
                    echo 'Checking out the repository...'
                }
                checkout scm
            }
        }

        stage('Verify Branch') {
            steps {
                script {
                    echo "Current Branch Name: '${env.BRANCH_NAME}'"
                    if (env.BRANCH_NAME == 'feature-ci-pipeline') {
                        echo "Executing pipeline for branch: ${env.BRANCH_NAME}"
                    } else {
                        echo "Skipping pipeline execution as the branch is not 'feature-ci-pipeline'"
                        currentBuild.result = 'ABORTED'
                        error("Stopping pipeline execution")
                    }
                }
            }
        }

        stage('Set up .NET Core') {
            steps {
                script {
                    echo 'Setting up .NET Core SDK...'
                }
                bat 'dotnet --version' // Ensure .NET is installed
            }
        }

        stage('Restore Dependencies') {
            steps {
                script {
                    echo 'Restoring dependencies...'
                }
                bat 'dotnet restore Homies.sln'
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Building the solution...'
                }
                bat 'dotnet build Homies.sln --no-restore'
            }
        }

        stage('Run Tests in Parallel') {
            parallel {
                stage('Run Unit Tests in Parallel') {
                    steps {
                        script {
                    echo 'Running Unit Tests...'
                }
                bat 'dotnet test Homies.Tests/Homies.Tests.csproj --no-build --verbosity normal'
            }
                }
                stage('Run Integration Tests in Parallel') {
                    steps {
                        script {
                    echo 'Running Integration Tests...'
                }
                bat 'dotnet test Homies.IntegrationTests/Homies.IntegrationTests.csproj --no-build --verbosity normal'
                }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}