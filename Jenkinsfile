pipeline {
   agent any
   environment {
      REPOSITORY = 'harbor.msalem.xyz'
      PCC_CONSOLE_URL = "172.16.50.4:8083"
      CONTAINER_NAME = "ubuntu"
   }
    stages {
         stage('Clone repository') {
            steps {
                checkout scm
            }
         }      

         stage('Build') {
            steps {
               withCredentials([usernamePassword(credentialsId: 'docker_registry_creds', passwordVariable: 'REGISTRY_PASS', usernameVariable: 'REGISTRY_USER')]) {                
                  sh ''' 
                  docker login -u $REGISTRY_USER -p $REGISTRY_PASS $REPOSITORY
                  echo "Building the Docker image..."
                  docker build -t $REPOSITORY/library/$CONTAINER_NAME:$BUILD_NUMBER .
                  docker image ls
                  '''
                }
            }
         }

         stage('Container Scan') {
            steps {
               script{
                  try {
                    prismaCloudScanImage ca: '', cert: '', dockerAddress: 'unix:///var/run/docker.sock', ignoreImageBuildTime: true, image: "$REPOSITORY/library/$CONTAINER_NAME:$BUILD_NUMBER", key: '', logLevel: 'debug', podmanPath: '', project: '', resultsFile: 'prisma-cloud-scan-results.json'
                  } finally {
                    prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
                  }
               }
            }
         }         

         stage('Push Image') {
            steps {
                  sh ''' 
                  echo "Image push into registry"
                  docker push $REPOSITORY/library/$CONTAINER_NAME:$BUILD_NUMBER
                  '''
            }
         }

         stage('Container Sandbox Scan') {
            steps {
               withCredentials([usernamePassword(credentialsId: 'ssh_creds', passwordVariable: 'SSH_PASS', usernameVariable: 'SSH_USER')]) {
                  sh '''         
                   sudo PCC_CONSOLE_URL=$PCC_CONSOLE_URL CONTAINER_NAME=$CONTAINER_NAME TAG=$BUILD_NUMBER /sandbox-scan.sh
                  '''
               }
            }
         }  
      }
   }
