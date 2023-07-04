pipeline {
    agent {label 'slave'}

    environment {
        function_name = 'test-java-app'
    }

    parameters {
        string(name: RollBackVersion, description: "Please enter rollback version")
        choice(
            choices: ['Dev', 'Test', 'Prod'],
            name: 'Environment',
            description: 'Please Select the environment to deploy to'
        )
    }

    stages {
        // CI Started

        stage('Build') {
            steps {
                echo 'Build'
                sh 'mvn package'
            }
        }

        stage('Sonar Analysis') {
            when {
                anyOf {
                    branch "feature/*"
                    branch "main"
                }
            }

            steps {
                echo 'Sonar Analysis'
                withSonarQubeEnv('Sonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Sonar Quality Gate') {
            steps {
                script {
                    try {
                        timeout(time: 10, unit: 'MINUTES') {
                            waitForQualityGate abortPipeline: true
                        }
                    }
                    catch (Exception ex) {

                    }
                }
            }
        }

        stage('Push') {
            steps {
                echo 'Push'

                // sh "aws s3 cp target/sample-1.0.3.jar s3://bermtecbatch31"
            }
        }

        // CI Ended

        // CD Started

        stage('Deployments') {
            parallel {
                stage('Deploy to Dev') {
                    steps {
                        echo 'Build'

                        // sh "aws lambda update-function-code --function-name $function_name --region us-east-1 --s3-bucket bermtecbatch31 --s3-key sample-1.0.3.jar"
                    }
                }

                stage('Deploy to Test') {
                    when {
                        branch "main"
                    }
                    steps {
                        echo 'Build'

                        // sh "aws lambda update-function-code --function-name $function_name --region us-east-1 --s3-bucket bermtecbatch31 --s3-key sample-1.0.3.jar"
                    }
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                expression { return params.Environment == 'Prod'}
            }
            steps {
                echo 'Deploying to Prod'
                input (
                    message: "Are we good for Prod Deployment?"
                )
            }
        }

        stage ('Release to Prod') {
            when {
                branch "main"
            }
            steps {
                echo "Release to Prod"
                // sh "aws lambda update-function-code --function-name $function_name --region us-east-1 --s3-bucket bermtecbatch31 --s3-key sample-1.0.3.jar"
            }
        }

        // CD Ended

    }

    post {
            always {
                echo "${env.BUILD_ID}"
                echo "${BRANCH_NAME}"
                echo "${BUILD_NUMBER}"
                echo "${JENKINS_URL}"
            }
            failure {
                echo "Failed to execute"
                mail(to:"tc.chawla2000@gmail.com", body:"This build failed. Please try again", subject:"Build Failure")
            }
            aborted {
                echo "Aborted"
            }
        }
}
