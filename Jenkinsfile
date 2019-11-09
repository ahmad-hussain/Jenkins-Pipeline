#!/usr/bin/env groovy

GoogleWebhook = 'Webhook URL'
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
                  sh"yarn test:ci"
                }
          }
          
          stage('Code analysis') { 	
          	def version = readFile ('version.txt')

    		if (!version) {
        		error("Version file (version.txt) was not found")
   			}

    		version = version.trim() +  "." + env.BUILD_NUMBER
                
            withSonarQubeEnv('Sonar Environment') {
      			sh "sonar-scanner -X -Dsonar.projectVersion=${version}"
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
                      input(message: 'Deploy this build to Production?')
                  }
              } catch (e) {
                  deployFlag = false
                  println "Not deploying to Production"
                  googlechatnotification message: "NOT deploying to Production", sameThreadNotification: true, url: GoogleWebhook;

              }
              if (deployFlag) {
                  googlechatnotification message: "*Deploying to QA*", sameThreadNotification: true, url: GoogleWebhook;
                try {
                	//deploy to Production
                } catch (owo) {
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
          googlechatnotification message: "Pipeline build ${BUILD_TAG} complete. *${currentBuild.currentResult}*. Link to build: ${BUILD_URL}", sameThreadNotification: true, url: GoogleWebhook;	
          junit testResults: 'junit.xml'
        }
    
}
