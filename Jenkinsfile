#!groovy
node {

   // Mark the code checkout 'stage'....
   stage 'Checkout'
   deleteDir()
   git credentialsId: '26968ab7-0a62-43a9-87a0-50132c074c09', url: 'https://github.com/rai-prashanna/springangular.git'

   // Mark the code build 'stage'....
   stage 'Build'
   // Run the maven build  added logic to send attach log inside email
   emailext attachLog: true, body: 'Check console output at http://jenkins.fusemachines.com/job/$JOB_NAME/$BUILD_NUMBER/ to view the results.', subject: '$BUILD_STATUS - $JOB_NAME -$BUILD_NUMBER', to: 'prashanna@fusemachines.com'
   
   try {
		   withEnv(['JAVA_TOOL_OPTIONS="-Dspring.profiles.active=prod"']) {
		   withMaven(maven: 'M2') {
           mvn clean install -DskipTests
       }
     }
  }
  
  catch(all){
      if(currentBuild.result!='FAILURE'){
         emailext attachLog: true, body: '$DEFAULT_CONTENT', subject: '$BUILD_STATUS - $JOB_NAME', to: 'prashanna@fusemachines.com'
      }
  	if(currentBuild.result == 'FAILURE'){
         emailext attachLog: true, body: '$DEFAULT_CONTENT', subject: '$BUILD_STATUS - $JOB_NAME', to: 'prashanna@fusemachines.com'
      }	
  }

	stage 'upload to s3'
		 // DEPLOY TO BEANSTALK
        def destinationWarFile = "code-challenge-${env.BUILD_NUMBER}.jar"
        def versionLabel = "code-challenge#${env.BUILD_NUMBER}"
        def description = "${env.BUILD_URL}"
   sh '''aws s3 cp /home/ec2-user/.jenkins/workspace/spring-pipeline/target/Spring4MVCAngularJSExample.war s3://jenkins-pipeline-test/
aws elasticbeanstalk create-application-version --source-bundle S3Bucket=jenkins-pipeline-test,S3Key=Spring4MVCAngularJSExample.war  --application-name Spring-boot --version-label v1 --description "just used for testing"
aws elasticbeanstalk update-environment --environment-name springtiles --application-name Spring-boot --version-label v1 --description "updated"
'''

        sleep 10L // wait for beanstalk to update the HealthStatus

        // WAIT FOR BEANSTALK DEPLOYMENT
        timeout(time: 5, unit: 'MINUTES') {
            waitUntil {    
                sh "aws elasticbeanstalk describe-environments --environment-names springtiles --attribute-names All > .beanstalk-status.json"
                // parse `describe-environment-health` output
                def beanstalkStatusAsJson = readFile(".beanstalk-status.json")
                def beanstalkStatus = new groovy.json.JsonSlurper().parseText(beanstalkStatusAsJson)
                println "$beanstalkStatus"
                return beanstalkStatus.HealthStatus == "Ok" && beanstalkStatus.Status == "Ready"
            }
        }
    }


		

  