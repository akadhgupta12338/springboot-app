pipeline {
    agent any
    
    tools
    {
       maven "Maven"
    }
     
    stages {
      stage('checkout') {
           steps {
                git branch: 'main', url: 'https://github.com/akadhgupta12338/springboot-app.git'
          }
        }
         
      stage('Execute Maven') {
           steps {
               sh 'mvn clean install'            
           }
        }
        
        stage("SonarQube analysis") {
            steps {
              withSonarQubeEnv('sonarqube') {
                sh 'mvn sonar:sonar'
              }
            }
          }
         
      stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
   
      stage('Jfrog Artifactory Upload'){
            steps{
                script{
                    def readpom;
                    readpom = readMavenPom file: '';
                    def artifactversion = readpom.version;
                rtUpload (
                 serverId:"artifactory" ,
                  spec: """{
                   "files": [
                      {
                      "pattern": "*.jar",
                      "target": "my-demo-libs-release-local/${artifactversion}/",
                      "recursive": "true"
                      }
                            ]
                           }""",
                        )
                }
            }
        }
        
      stage ('Jfrog Artifactory Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "artifactory"
                )
            }
        }
        
      stage('Building and pusing image to ECR') {
            steps{
              script {
            sh 'docker build -t my-docker-repo .'
            sh 'docker tag my-docker-repo:latest 737376305814.dkr.ecr.ap-south-1.amazonaws.com/my-docker-repo:latest'
            sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 737376305814.dkr.ecr.ap-south-1.amazonaws.com'
            sh 'docker push 737376305814.dkr.ecr.ap-south-1.amazonaws.com/my-docker-repo:latest'
             }
          }
      }
      
      stage('eks deploy')
      {
          steps{
              withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'eks-demo-k8s', namespace: '', serverUrl: ''){
                sh "kubectl apply -f eks-deploy-from-ecr.yaml"
              }
          }
       }   
    }
}
