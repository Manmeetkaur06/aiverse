pipeline {
    agent any
    
    triggers {
        githubPush()  
    }

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-21-openjdk-amd64"
        MAVEN_HOME = "/usr/share/maven"
        AWS_CLI_PATH = "/usr/bin"
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${AWS_CLI_PATH}:$PATH"
        SONARQUBE_CREDENTIALS = 'sonarqube'
        S3_BUCKET = 'mypipelinebucket'
        APP_NAME = 'aiverse'
        ENV_NAME = 'Aiverse-env'

        
        EMAIL_RECIPIENTS = "1999manmeetkaur@gmail.com"
    }

    stages {
        stage('Build') {
            steps {
                script {
                    sh 'rm -rf *'
                    git branch: 'main', url: 'https://github.com/Manmeetkaur06/aiverse.git'
                    sh 'java -version'
                    sh 'mvn --version'
                    sh 'mvn clean package -DskipTests -Dmaven.compiler.release=21'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh 'mvn test'
                }
                junit '**/target/surefire-reports/*.xml'
            }
        }

        stage('Analysis') {
            steps {
                withSonarQubeEnv('Sonarqube') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=Manmeetkaur06_aiverse \
                          -Dsonar.organization=manmeetkaur06 \
                          -Dsonar.host.url=https://sonarcloud.io \
                          -DskipTests
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                withAWS(credentials: 'aws-cred', region: 'eu-north-1') {
                    script {
                        sh 'aws s3 cp target/aiverse-0.0.1-SNAPSHOT.jar s3://${S3_BUCKET}/aiverse-${BUILD_NUMBER}.jar'
                        
                        sh '''
                            aws elasticbeanstalk create-application-version \
                              --application-name ${APP_NAME} \
                              --version-label v${BUILD_NUMBER} \
                              --source-bundle S3Bucket=${S3_BUCKET},S3Key=aiverse-${BUILD_NUMBER}.jar
                        '''
                        
                        sh '''
                            aws elasticbeanstalk update-environment \
                              --environment-name ${ENV_NAME} \
                              --version-label v${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo "📧 Sending email notification..."
        }
        success {
            emailext(
                subject: "✅ Jenkins Build #${BUILD_NUMBER} Successful!",
                body: """
                    The Jenkins build was successful. 🎉
                    \n🔗 Job: ${env.JOB_NAME}
                    \n🔢 Build: #${env.BUILD_NUMBER}
                    \n🔗 View Build: ${env.BUILD_URL}
                    \n✅ Deployment was successful!
                """,
                to: "${EMAIL_RECIPIENTS}"
            )
        }
        failure {
            emailext(
                subject: "❌ Jenkins Build #${BUILD_NUMBER} Failed!",
                body: """
                    The Jenkins build **FAILED**! ❌
                    \n🔗 Job: ${env.JOB_NAME}
                    \n🔢 Build: #${env.BUILD_NUMBER}
                    \n🔗 View Build Logs: ${env.BUILD_URL}
                    \n❗ Check the logs to fix the issue.
                """,
                to: "${EMAIL_RECIPIENTS}"
            )
        }
    }
}
