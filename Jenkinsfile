pipeline {
    agent any
    environment {
        // SonarQube
        SONAR_QUBE_CREDENTIALS_ID = 'sonar-token'
        SONAR_QUBE_NAME = 'sonarqube_server'

        // Nexus
        NEXUS_REPOSITORY_ID = 'Nexus_customer_app'
        NEXUS_URL = 'http://98.82.189.119:8081//repository/Nexus_customer_app/'

        // Tomcat
        TOMCAT_URL = 'http://34.202.205.194:8080/manager/text'
        TOMCAT_CREDENTIALS_ID = 'tomcat-credentials'
        TOMCAT_APP_CONTEXT = 'SimpleCustomerApp'
    }

    tools {
        maven 'MVN_HOME'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'feature-1.1', url: 'https://github.com/sushma-0611/sabear_simplecutomerapp.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(credentialsId: "${SONAR_QUBE_CREDENTIALS_ID}", installationName: "${SONAR_QUBE_NAME}") {
                    sh 'mvn clean verify sonar:sonar -DskipTests'
                }
            }
        }

        stage('Maven Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-credentials',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    writeFile file: 'settings-temp.xml', text: """
                        <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0">
                          <servers>
                            <server>
                              <id>${env.NEXUS_REPOSITORY_ID}</id>
                              <username>${env.NEXUS_USER}</username>
                              <password>${env.NEXUS_PASS}</password>
                            </server>
                          </servers>
                        </settings>
                    """
                    sh 'mvn deploy -DskipTests --settings settings-temp.xml'
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def originalWar = 'target/SimpleCustomerApp-1.0.0-SNAPSHOT.war'
                    def renamedWar = "target/${env.TOMCAT_APP_CONTEXT}.war"

                    if (fileExists(originalWar)) {
                        sh "cp ${originalWar} ${renamedWar}"

                        step([
                            $class: 'DeployPublisher',
                            adapters: [[
                                $class: 'Tomcat9xAdapter',
                                credentialsId: "${TOMCAT_CREDENTIALS_ID}",
                                url: "${TOMCAT_URL}"
                            ]],
                            war: renamedWar,
                            contextPath: "${env.TOMCAT_APP_CONTEXT}"
                        ])
                    } else {
                        error "WAR file not found at ${originalWar}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }

        success {
            echo ':white_check_mark: Pipeline succeeded.'
            slackSend(
                channel: '#jenkins-integration',
                color: 'good',
                message: "*✅ SUCCESS*: Build ${env.BUILD_NUMBER} for *${env.JOB_NAME}* succeeded!",
                tokenCredentialId: 'Slack-Token'
            )
        }

        failure {
            echo ':x: Build or Deployment Failed!'
            slackSend(
                channel: '#jenkins-integration',
                color: 'danger',
                message: "*❌ FAILURE*: Build ${env.BUILD_NUMBER} for *${env.JOB_NAME}* failed!",
                tokenCredentialId: 'Slack-Token'
            )
        }
    }
}
