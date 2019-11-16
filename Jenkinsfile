#!/usr/bin/env groovy
@Library('Jenkins-Library')_
//importing library, make sure it is setup on jenkins first

googleWebhook = 'Webhook URL'
//Webhook for jenkins bot 
message = "*ACTION REQUIRED* Pipeline build ${BUILD_TAG} requesting input to deploy to *PRODUCTION*. Click _'Proceed'_  to deploy, click _'Abort'_  to skip deployment. Link to build: ${BUILD_URL}console"

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
		  sonarCodeAnalysis()		  
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
			googleChat([Message: message, GoogleWebhook: googleWebhook])			  
			  //send a message to the google chat room asking for user input
                      input(message: 'Deploy this build to Production?')
			  //asks for user input with options  'Proceed' or 'Abort' (those are the default) 
                  }
              } catch (e) {
		      //if no input provided before timeout then do this
                  deployFlag = false
                  println "Not deploying to Production"
                  message = "NOT deploying to Production"
		  googleChat([Message: message, GoogleWebhook: googleWebhook])

              }
              if (deployFlag) {
		      //send this message if user says proceed
                  message = "*Deploying to Production*"
		   googleChat([Message: message, GoogleWebhook: googleWebhook])
                try {
                	//deploy to Production
                } catch (owo) {
			//do this if deployment step somehow fails 
                  currentBuild.result = "UNSTABLE"
                  message = "Failed to deploy to Production"
			googleChat([Message: message, GoogleWebhook: googleWebhook])	
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
          message = "Pipeline build ${BUILD_TAG} complete. *${currentBuild.currentResult}*. Link to build: ${BUILD_URL}"
	  googleChat([Message: message, GoogleWebhook: googleWebhook])	
          junit testResults: 'junit.xml'
	    //read the generated junit.xml for that fancy looking test trend graph
        }
    
}
