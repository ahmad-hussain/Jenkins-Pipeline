#!/usr/bin/env groovy

GoogleWebhook = 'Webhook URL'
//Webhook for jenkins bot 
GoogleChatApproveDeploymentMessage = "*ACTION REQUIRED* Pipeline build ${BUILD_TAG} requesting input to deploy to *PRODUCTION*. Click _'Proceed'_  to deploy, click _'Abort'_  to skip deployment. Link to build: ${BUILD_URL}console"

node('Slave-Node') {
  String OUTPUT_DIR = pwd()
    try {
          
			stage('Clone code') {
				checkout scm
			}
		
			stage('Install dependancies'){
            	nodejs('new node') {
                	sh"yarn install"
				}
			}
          
          stage('Run component tests') {
          		nodejs('new node') {
                  sh"CI=TRUE yarn test"
		//run some unit tests which were dependent on react testing library, not covered by test:ci script
                  sh"yarn test:ci"
		//generates test reports and coverage reports for Junit test result graph and sonar code coverage 
                }
          }
          
          stage('Code analysis') { 	
          	def version = readFile ('version.txt')
		  //include a .txt file with just project version (e.g '1.0.3') in project folder 

    		if (!version) {
        		error("Version file (version.txt) was not found")
   			}

    		version = version.trim() +  "." + env.BUILD_NUMBER
                //appends version with the build number
            withSonarQubeEnv('Sonar Environment') {
      			sh "sonar-scanner -X -Dsonar.projectVersion=${version}"
		    //adds the version number to sonar so you can easily see differences between builds. Make sure to have a seperate sonar-project.properties
    		} 
          }
          
        stage('Deploy to QA') {
          sh"ssh user@QA_IP" //followed by deployment steps
        }
        
        stage('Run integration Tests') {
          //run integration tests
        }
        
        stage('Deploy to staging') {
          //deploy to staging
         }
         
         stage('End to end tests') {
          //run end to end tests
         }

                       
          stage('Deploy to PROD') {
            node('master') {
            def deployFlag = true
              try {
                  timeout(time: 1, unit: "HOURS") {
                      googlechatnotification message: GoogleChatApproveDeploymentMessage, sameThreadNotification: true, url: GoogleWebhook;
			  //send a message to the google chat room asking for user input
                      input(message: 'Deploy this build to Production?')
			  //asks for user input with options  'Proceed' or 'Abort' (those are the default) 
                  }
              } catch (e) {
		      //if no input provided before timeout then do this
                  deployFlag = false
                  println "Not deploying to Production"
                  googlechatnotification message: "NOT deploying to Production", sameThreadNotification: true, url: GoogleWebhook;

              }
              if (deployFlag) {
		      //send this message if user says proceed
                  googlechatnotification message: "*Deploying to Production*", sameThreadNotification: true, url: GoogleWebhook;
                try {
                	//deploy to Production
                } catch (owo) {
			//do this if deployment step somehow fails 
                  currentBuild.result = "UNSTABLE"
                  googlechatnotification message: "Failed to deploy to Production", sameThreadNotification: true, url: GoogleWebhook;
                }
              } else {
                  println "Something very unusual must have happened"
              }
          }
        }
          
          
    } catch (err) {	
	   currentBuild.result = 'FAILURE'
        } finally {
	    //send the build result to google chat. N.B- can't define this message at the top cause it'll mess up the build result
          googlechatnotification message: "Pipeline build ${BUILD_TAG} complete. *${currentBuild.currentResult}*. Link to build: ${BUILD_URL}", sameThreadNotification: true, url: GoogleWebhook;	
          junit testResults: 'junit.xml'
	    //read the generated junit.xml for that fancy looking test trend graph
        }
    
}
