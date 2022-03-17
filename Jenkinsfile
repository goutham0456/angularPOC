pipeline {
     agent any
     environment {
        dockerImage = ''
        registry = 'ashish0786/angularuiapp'
        registryCredential ='dockerhub_id'
    }
    stages {
        stage ('checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github_id', url: 'https://github.com/ashish0626/angularPOC.git']]])
            }
        }
         stage('Scan') {
            steps {
                    withSonarQubeEnv(installationName: 'sq1') { 
                    sh './mvnw clean org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.0.2155:sonar'
                }
             }
        }
        stage("Quality Gate") {
            steps {
                    timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build Docker Images') {
            steps {
                script {
                    dockerImage = docker.build registry
                }
            }
        }
        stage('Upload to DockerHub'){
            steps {
                script {
                        docker.withRegistry('',registryCredential){
                            dockerImage.push()
                        }
                }
            }
        }
        stage('docker stop container') {
         steps {
            sh 'docker ps -f name=practical_newton  -q | xargs --no-run-if-empty docker container stop'
            sh 'docker container ls -a -fname=practical_newton -q | xargs -r docker container rm'
         }
       }
       stage('Docker Run') {
        steps {
            script {
                dockerImage.run("-p 8019:80 --rm --name practical_newton")
                }
            }
        }
    }
}
