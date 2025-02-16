node{
    def mavenHome = tool name: "maven3.9.9"
    def buildNumber = BUILD_NUMBER

// To keep last 5 buil only, old one will be delete
        properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5', removeLastBuild: true)), pipelineTriggers([])])

// Github will notify to jenkins once changes/commit is done in github and jenkins starts build automatically 
       properties([pipelineTriggers([githubPush()])])

	
    // Checkout stage
    stage('Git Clone'){
        git branch: 'master', credentialsId: 'GithubCred', url: 'https://github.com/TheAmitDeokar/spring-boot-mongo-docker.git'
    }
    
    // Build stage
   	stage('Build app package'){
	sh "$mavenHome/bin/mvn clean package"
         }
      
     // Build Docker Image
    stage('Build Docker image'){
    sh "docker build -t 686255940829.dkr.ecr.ap-south-1.amazonaws.com/spring-boot-mongo-docker:${buildNumber} ."
    } 
    
    // Docker Login and push image
    stage('Docker Login and push image'){
    
   sh "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 686255940829.dkr.ecr.ap-south-1.amazonaws.com"       

   sh "docker push 686255940829.dkr.ecr.ap-south-1.amazonaws.com/spring-boot-mongo-docker:${buildNumber}"
    }
    
    // Deploy App as Docker Container in Docker Deployment server
     stage('Deploy App as Docker Container in Docker Deployment server'){
        sshagent(['ubuntudeploymentserver']) {

          sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.43.211 docker rm -f springbootappcontainer || true"

          sh "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 686255940829.dkr.ecr.ap-south-1.amazonaws.com" 
          // mongo db as a container
          sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.43.211 docker run -d --name mongo -e MONGO_INITDB_ROOT_USERNAME:root -e MONGO_INITDB_ROOT_PASSWORD:passw0rd mongo"

          sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.43.211 docker run -d --name springbootappcontainer -e MONGO_DB_HOSTNAME:mongo -e MONGO_DB_USERNAME:mongo -e MONGO_DB_PASSWORD:passw0rd -p 8081:8080 686255940829.dkr.ecr.ap-south-1.amazonaws.com/spring-boot-mongo-docker:${buildNumber}"
  
    }
     }

}