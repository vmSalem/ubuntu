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
                  sudo docker login -u $REGISTRY_USER -p $REGISTRY_PASS $REPOSITORY
                  echo "Building the Docker image..."
                  sudo docker build -t $REPOSITORY/library/$CONTAINER_NAME:$BUILD_NUMBER .
                  sudo docker image ls
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
                   mkdir -p ~/.ssh/
                   ssh-keyscan -t rsa,dsa 10.160.154.170 >> ~/.ssh/known_hosts
                   sshpass -p $SSH_PASS ssh $SSH_USER@10.160.154.170 'bash -s' <<EOF         
                   sudo chmod +x /home/sysadmin/apps/sandbox-scan.sh
                   sudo PCC_CONSOLE_URL=$PCC_CONSOLE_URL token=$token CONTAINER_NAME=$CONTAINER_NAME TAG=$BUILD_NUMBER /home/sysadmin/apps/sandbox-scan.sh
                   exit
                   EOF
                  '''
               }
            }
         }  
      }
   }
