pipeline {
    agent any

    stages {
        stage('Git checkout') {
            steps{
                git branch: 'master', credentialsId: 'f41ee166-3661-42e2-a229-84cb2bd96ad0', url:'git@github.com:NikitaZhevnitskiy/devops-exam.git'
            }
        }
        stage('Test')  {
            tools {
                jdk "JDK 8"
                maven "maven-3.3.9"
            }
            steps {
                sh('mvn clean')
                sh('mvn install')
            }
        }
        stage('Checkstyle report'){
            tools {
                maven "maven-3.3.9"
            }
            steps{
                sh('mvn checkstyle:checkstyle')
            }
            post {
                success {
                    step([$class: 'hudson.plugins.checkstyle.CheckStylePublisher', pattern: '**/target/checkstyle-result.xml', unstableTotalAll:'200'])
                }
            }
        }
        stage('FindBugs report'){
            tools {
                maven "maven-3.3.9"
            }
            steps{
                sh('mvn findbugs:findbugs')
            }
            post {
                success {
                    step([$class: 'FindBugsPublisher', pattern: '**/findbugsXml.xml', unstableTotalAll:'10'])
                }
            }
        }
        stage('Package'){
            tools {
                jdk "JDK 8"
                maven "maven-3.3.9"
            }
            steps {
                sh('mvn clean')
                sh('mvn package')
            }
            post {
                success {
                    echo 'Archiving...'
                    archiveArtifacts artifacts: '**/target/calculator.jar'
                }
            }
        }

        stage('Smoke test'){
            tools {
                jdk "JDK 8"
            }
            steps {
                timeout(2){
                    sh 'nohup java -jar -Dserver.port=8081 ./target/calculator.jar > /dev/null 2>&1&'
                    sh 'while ! curl http://localhost:8081/health-check ; do sleep 5;  done'
                }
            }
            post {
                success {
                    sh 'curl -X POST http://localhost:8081/shutdown'
                }
                failure {
                    echo "Fail"
                }
            }
        }
    }
}