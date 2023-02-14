pipeline {
    triggers {
  pollSCM('* * * * *')
    }
   agent any
    tools {
  maven 'M2_HOME'
}
environment {
    registry = '076892551558.dkr.ecr.us-east-1.amazonaws.com/jenkins'
    registryCredential = 'aws_ecr_id'
    dockerimage = ''

    // NEXUS_VERSION = "nexus3"
    // NEXUS_PROTOCOL = "http"
    // NEXUS_URL = "139.177.192.139:8081"
    // NEXUS_REPOSITORY = "utrains-nexus-pipeline"
    // NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
}

// environment {
       

//         imageName = "fastfood"
//         registryCredentials = "nexus-user-credentials"
//         registry = "139.177.192.139:8085/repository/utrains-nexus-registry/"
//         dockerImage = ''

//         //Declare the variable version
//         POM_VERSION = ''
//         BUILD_NUM = "${env.BUILD_ID}"

//     }

    stages {

        stage("build & SonarQube analysis") {
            agent {
        docker { image 'maven:3.8.6-openjdk-11-slim' }
   }
            
            
            steps {
                echo 'build & SonarQube analysis...'
            //   withSonarQubeEnv('SonarServer') {
            //       sh 'mvn sonar:sonar -Dsonar.projectKey=kserge2001_geolocation -Dsonar.java.binaries=.'
            //   }
            }
          }
        stage('Check Quality Gate') {
            steps {
                echo 'Checking quality gate...'
                // script {
                //     timeout(time: 20, unit: 'MINUTES') {
                //         def qg = waitForQualityGate()
                //         if (qg.status != 'OK') {
                //             error "Pipeline stopped because of quality gate status: ${qg.status}"
                //         }
                //     }
                // }
            }
        }
        
         
        stage('maven package') {
            steps {
                sh 'mvn clean'
                sh 'mvn install -DskipTests'
                sh 'mvn package -DskipTests'
            }
        }
        stage('Build Image') {
            
            steps {
                script{
                  def mavenPom = readMavenPom file: 'pom.xml'
                    dockerImage = docker.build registry + ":${mavenPom.version}"
                } 
            }
        }
        stage('Deploy image') {
           
            
            steps{
                script{ 
                    docker.withRegistry("https://"+registry,"ecr:us-east-1:"+registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        } 

        //Project Helm Chart push as tgz file
        stage("pushing the Backend helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus-pass', variable: 'docker_password')]) {
                       
                        sh '''
                            helmversion=$( helm show chart app | grep version | cut -d: -f 2 | tr -d ' ')
                            tar -czvf  app-${helmversion}.tgz app/
                            curl -u jenkins-user:$docker_password http://139.177.192.139:8081/repository/geolocation/ --upload-file app-${helmversion}.tgz -v
                        '''
                    }
                }
            }
        }   
         
         
    }
}
