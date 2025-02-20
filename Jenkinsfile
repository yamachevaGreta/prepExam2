pipeline {
    agent any

    environment {
        DOTNET_VERSION = '6.0.x' // Define .NET version for consistency
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
                    // Detect the current branch name
                    def branchName = env.BRANCH_NAME ?: env.GIT_BRANCH ?: 'unknown'
                    echo "Detected branch: '${branchName}'"

                    // Allow execution only on the 'feature-ci-pipeline' branch
                    if (branchName.endsWith('feature-ci-pipeline') || branchName == 'feature-ci-pipeline') {
                        echo "Executing pipeline for branch: ${branchName}"
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
                bat 'dotnet --version' // Ensure .NET is installed and accessible
            }
        }

        stage('Restore Dependencies') {
            steps {
                script {
                    echo 'Restoring dependencies...'
                }
                bat 'dotnet restore Homies.sln' // Restore NuGet packages
            }
        }

        stage('Validate Code Format') {
            steps {
                script {
                    echo 'Validating code format using dotnet format...'
                }
                bat 'dotnet format --verify-no-changes || exit 1' // Ensure code follows formatting standards
            }
        }

        stage('Lint Code & Syntax Check') {
            steps {
                script {
                    echo 'Running syntax check and code linting...'
                }
                bat 'dotnet validate' // Validate project structure and syntax
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Building the solution...'
                }
                bat 'dotnet build Homies.sln --no-restore' // Build without restoring dependencies again
            }
        }

        stage('Run Tests in Parallel') {
            parallel {
                stage('Run Unit Tests') {
                    steps {
                        script {
                            echo 'Running Unit Tests...'
                        }
                        bat 'dotnet test Homies.Tests/Homies.Tests.csproj --no-build --verbosity normal'
                    }
                }
                stage('Run Integration Tests') {
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
