node {
    def mavenHome, mavenCMD, docker, tag, dockerHubUser, containerName, httpPort = ""
    
    stage('Prepare Environment') {
        echo 'Initialize Environment'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        tag="latest"
        dockerHubUser="nileshk12"
        containerName="insure"
        httpPort="8081"
    }
    
    stage('Code Checkout') {
        try {
            checkout scm
        } catch(Exception e) {
            echo 'Exception occurred in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            //emailext body: '''Dear All,
            //The Jenkins job ${JOB_NAME} has been failed. Request you to please have a look at it immediately by clicking on the below link. 
            //${BUILD_URL}''', subject: 'Job ${JOB_NAME} ${BUILD_NUMBER} is failed', to: 'jenkins@gmail.com'
        }
    }
    
    stage('Maven Build') {
        sh "mvn clean package"        
    }
    
    stage('Docker Image Build') {
        echo 'Creating Docker image'
        sh "docker build -t $dockerHubUser/$containerName:$tag --pull --no-cache ."
    }
    
    stage('Docker Image Scan') {
        echo 'Scanning Docker image for vulnerabilities'
        sh "docker build -t ${dockerHubUser}/insure-me:${tag} ."
    }   
    
    stage('Publishing Image to DockerHub') {
        echo 'Pushing the docker image to DockerHub'
        withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
            sh "docker login -u $dockerUser -p $dockerPassword"
            sh "docker push $dockerUser/$containerName:$tag"
            echo "Image push complete"
        } 
    }    
    
    stage('Docker Container Deployment') {
        sh "docker rm $containerName -f"
        sh "docker pull $dockerHubUser/$containerName:$tag"
        sh "docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
        echo "Application started on port: ${httpPort} (http)"
    }
}
