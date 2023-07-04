pipeline {
    agent {label 'slave'}

    environment {
        function_name = 'java-sample'
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
                timeout(time:10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Push') {
            steps {
                echo 'Push'

                sh "aws s3 cp target/sample-1.0.3.jar s3://bermtecbatch31"
            }
        }

        // CI Ended

        // CD Started

        stage('Deployments') {
            parallel {
                stage('Deploy to Dev') {
                    steps {
                        echo 'Build'

                        sh "aws lambda update-function-code --function-name $function_name --region us-east-1 --s3-bucket bermtecbatch31 --s3-key sample-1.0.3.jar"
                    }
                }

                stage('Deploy to Test') {
                    steps {
                        echo 'Build'

                        sh "aws lambda update-function-code --function-name $function_name --region us-east-1 --s3-bucket bermtecbatch31 --s3-key sample-1.0.3.jar"
                    }
                }
            }
        }

        stage('Deploy to Prod') {
            steps {
                echo 'Build'

                sh "aws lambda update-function-code --function-name $function_name --region us-east-1 --s3-bucket bermtecbatch31 --s3-key sample-1.0.3.jar"
            }
        }

        // CD Ended
    }
}
