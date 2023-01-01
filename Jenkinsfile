node('nodes'){

    try{    
    def mavenHome = tool name: 'maven3.8.5'
    
    echo "The Job name is: ${env.JOB_NAME}"
    echo "The Build number is: ${env.BUILD_NUMBER}"
    echo "The Node name is: ${env.NODE_NAME}"

properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([pollSCM('* * * * *')])])    
    
//Checkout Code State
stage('CheckoutCode'){
    sendSlackNotifications("STARTED")
git branch: 'development', url: 'https://github.com/subhendu1994/maven-web-application.git'
}

//Build
stage('Build'){
sh "${mavenHome}/bin/mvn clean package"
}


//Execute SonarQube Report
stage('ExecuteSonarQubeReport'){
sh "${mavenHome}/bin/mvn sonar:sonar"
}

//UploadArtifacts Into Nexus
stage('UploadArtifactsIntoNexus'){
sh "${mavenHome}/bin/mvn deploy"
}

//Deploy App into Tomcat server
stage('DeployAppp'){
sshagent(['3d62ecdc-0774-4476-8b2b-8a28b709ba38']) {
   sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@172.31.3.27:/opt/apache-tomcat-9.0.70/webapps"
}
}
}//try closing
catch(e){
currentBuild.result = "FAILURE"
}
finally{
sendSlackNotifications(currentBuild.result)
}

}//node closing

//Functions for slack notifications

def sendSlackNotifications(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
