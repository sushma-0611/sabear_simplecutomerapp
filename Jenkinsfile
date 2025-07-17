pipeline {
    agent any

    environment {
        SONARQUBE_URL = 'http://http://3.81.41.198:9000/'
        SONAR_QUBE_NAME = 'sonarqube-server'
        NEXUS_REPO = 'Nexus_customer_app'
        
    }
    tools{
        maven 'MVN_HOME'
    }
    stages {

        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/sushma-0611/sabear_simplecutomerapp.git'
            }
        }

        stage('Sonarqube Integration') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                            mvn sonar:sonar \
                                -Dsonar.projectKey=sabear_simplecutomerapp \
                                -Dsonar.host.url=${SONARQUBE_URL} \
                                -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Maven Compilation') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Nexus Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                        mvn deploy \
                            -DaltDeploymentRepository=${NEXUS_REPO}::default::http://98.82.189.119:8081/repository/${NEXUS_REPO}/ \
                            -Dnexus.username=${NEXUS_USER} \
                            -Dnexus.password=${NEXUS_PASS}
                    """
                }
            }
        }

        stage('Slack Notification') {
            steps {
                slackSend channel: '#jenkins-integration', message: "Build Pipeline Completed for sabear_simplecutomerapp"
            }
        }
    }
}
