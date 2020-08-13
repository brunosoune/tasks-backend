pipeline {
    agent any
    stages {
        stage ('Build backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit tests') {
            steps {
                bat 'mvn test'
            }
        }
        stage ('Sonar analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=75f8c7a97a16bd99cc8534e6ab164b1c03e2ad72 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage ('Quality gate') {
            steps {
                sleep(5)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy backend') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API test') {
            steps {
                dir('api-test') {
                    git 'https://github.com/brunosoune/tasks-api-test'
                    bat 'mvn test'
                }
            }
        }
        stage ('Deploy frontend') {
            steps {
                dir('frontend') {
                    git 'https://github.com/brunosoune/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat9(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001')], contextPath: 'tasks', war: 'target/tasks.war'
                }
                
            }
        }
        stage ('Functional test') {
            steps {
                dir('functional-test') {
                    git 'https://github.com/brunosoune/tasks-functional-tests'
                    bat 'mvn test'
                }
            }
        }
    }
}