pipeline {
    agent any
    stages{
        stage ('Build Backend'){
            steps{
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests'){
            steps{
                bat 'mvn test'
            }
        }
        stage ('Sonar Analysis'){
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps{
                withSonarQubeEnv('SONAR_LOCAL') {
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=8eeee5e5e342d3bc4a5fab2aa3d89ac42390bff9 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/modal/**,**Application.java"
                }
            }
        }
        stage ('Quality Gate'){
            steps{
                sleep(30)
                timeout(time: 1, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        } 
        stage ('Deploy Backend'){
            steps{
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test'){
            steps{
                dir('api-test'){
                    git credentialsId: 'github_login', url: 'https://github.com/randellr7/tasks-api-test'
                    bat 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend'){
            steps{
                dir('frontend'){
                    git credentialsId: 'github_login', url: 'https://github.com/randellr7/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Functional Test'){
            steps{
                dir('functional-test'){
                    git credentialsId: 'github_login', url: 'https://github.com/randellr7/tasks-functional-test'
                    bat 'mvn test'
                }
            }
        }
        stage ('Deploy Prod'){
            steps{
                    bat 'docker-compose build'
                    bat 'docker-compose up -d'
            }
        }
         stage ('Health Check'){
            steps{
                sleep(20)
                dir('functional-test'){
                    bat 'mvn verify -Dskip.surefire.tests'
                }
            }
        }
    }
}







