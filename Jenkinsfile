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
    registryCredential = 'jenkins-ecr'
    dockerimage = ''

     NEXUS_VERSION = "nexus3"
     NEXUS_PROTOCOL = "http"
     NEXUS_URL = "139.177.192.139:8081"
     NEXUS_REPOSITORY = "utrains-nexus-pipeline"
     NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
     POM_VERSION = ''
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
            steps {
                echo 'build & SonarQube analysis...'
               withSonarQubeEnv('SonarServer') {
                   sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=Hermann90_geo -X'
               }
            }
          }
        stage('Check Quality Gate') {
            steps {
                echo 'Checking quality gate...'
                 script {
                     timeout(time: 20, unit: 'MINUTES') {
                         def qg = waitForQualityGate()
                         if (qg.status != 'OK') {
                             error "Pipeline stopped because of quality gate status: ${qg.status}"
                         }
                     }
                 }
            }
        }
        
         
        stage('maven package') {
            steps {
                sh 'mvn clean'
                sh 'mvn package -DskipTests'
            }
        }
        stage('Build Image') {
            
            steps {
                script{
                  def mavenPom = readMavenPom file: 'pom.xml'
                  POM_VERSION = "${mavenPom.version}"
                  echo "${POM_VERSION}"
                  dockerImage = docker.build registry + ":${POM_VERSION}"
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

        Project Helm Chart push as tgz file
        stage("pushing the Backend helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus-pass', variable: 'NexusID')]) {
                       
                        sh '''
                            helmversion=$( helm show chart app | grep version | cut -d: -f 2 | tr -d ' ')
                            tar -czvf  app-${helmversion}.tgz app/
                            curl -u jenkins-user:$NexusID http://139.177.192.139:8081/repository/geolocation/ --upload-file app-${helmversion}.tgz -v
                        '''
                    }
                }
            }
        }   
         
         
    }
}
