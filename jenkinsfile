pipeline {
    agent any
    tools {
        jdk 'java21'
        maven 'maven3.9.16'
    } 
    stages{
        stage ('checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/lokeshdevops01/Employee-Management-App.git'
            }
        }
        stage ('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage ('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube'){
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=employee-management-app\
                    -Dsonar.projectName=employee-management-app
                    '''
                }
            }
        }
        stage ('Deploy to Nexus') {
            steps {
                withCredentials ([
                    usernamePassword(
                        credentialsId: 'nexus-creds',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS'
                        )
                    ]) {
                        sh 'mvn deploy -s /var/lib/jenkins/.m2/settings.xml'
                }
            }
        }
        stage ('Docker Build') {
            steps {
                sh '''
                docker build \
                -t lokeshdevops01/employee-management-app:${BUILD_NUMBER} .
                '''
            }
        }
        stage ('Docker Push') {
            steps {
                sh '''
                docker push \
                lokeshdevops01/employee-management-app:${BUILD_NUMBER}
                '''
            }
        }
        stage ('Deploy to EC2 using ansible') {
            steps {
                    sh """
                    cd /var/lib/jenkins/ansible
                    
                    ansible-playbook -i inventory.ini \
                    -e build_number=${BUILD_NUMBER} \
                    -e previous_build=${previousBuild} \
                    ec2.yml
                    """
            }
        }
    }
}
