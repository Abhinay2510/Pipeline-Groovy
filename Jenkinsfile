import com.cloudbees.hudson.plugins.modeling.*
pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M2_HOME"
    }
	
    stages {
	   
            stage('load parameters') { 
            steps {
                load "parameters.groovy"
            }
        }	
	
        stage('SAST Analysis') {
            steps {
                script {
			def stageResults = [:] // Define the stageResults map

                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    try {
                // Get some code from a GitHub repository
						if (env.SAST_GIT_URL != '') {				
                git "${env.SAST_GIT_URL}"
                bat "git clone https://github.com/harishpallapu/sonarqube_scannar_windows.git"
                bat "./sonarqube_scannar_windows/sonar-scanner-4.6.2.2472-windows/bin/sonar-scanner.bat"
		stageResults['SAST Analysis'] = true
						} else {
                                                        echo "skipping the stage ${env.STAGE_NAME}.............................!"
							
                   } } 
			catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed...............!"
							stageResults['SAST Analysis'] = false
						}                
                // Run Maven on a Unix agent.
                //sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        } 
    }
}
    
       stage('SCA') {
    steps {
            script {
		    def stageResults = [:] // Define the stageResults map

                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                try {
                    def dependencyCheckZipUrl = 'https://github.com/jeremylong/DependencyCheck/releases/download/v6.1.5/dependency-check-6.1.5-release.zip'
                    def dependencyCheckZipPath = "${WORKSPACE}\\dependency-check-6.1.5-release.zip"
                    def extractionPath = "${WORKSPACE}\\dependency-check-6.1.5-release"

                    // Download the DependencyCheck release zip using curl
                    bat "curl -o ${dependencyCheckZipPath} ${dependencyCheckZipUrl}"

                    // Extract the downloaded zip using PowerShell
                    bat "PowerShell Expand-Archive -Path \"${dependencyCheckZipPath}\" -DestinationPath \"${extractionPath}\""

                    // Run DependencyCheck
                    bat "\"${extractionPath}\\dependency-check\\bin\\dependency-check.bat\" --noupdate --project \"TeachersFCU\" --scan \"Shoppingcart/lib/\" --format HTML --out \"${WORKSPACE}\""

                    // Mark the stage as successful
                    currentstage.resultIsSuccess = true
                } catch (Exception err) {
                    echo "Error: ${err.getMessage()}"
                    unstable(message: "${STAGE_NAME} is unstable")
                    echo "Error detected, ${env.STAGE_NAME} failed..."
                    
                    // Mark the stage as failed
                    currentstage.resultIsSuccess = false
                }
            }
        }
    }
}

	stage('build') {
            steps {
		
				script {
					def stageResults = [:] // Define the stageResults map

                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
						try {
						if (env.BUILD_GIT_URL != '') {
                git "${env.BUILD_GIT_URL}"
							bat 'mvn clean package'
						currentStage.resultIsSuccess = true	
					        } else {
                                                        echo 'skipping the stage ${env.STAGE_NAME}.............................!'
							currentStage.resultIsSuccess = false
                                                        }							
						} catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed..................!"
						
						}
				}							
            }
	 }
	}
     stage("deploy") {
         steps {
		  
					script {
						def stageResults = [:] // Define the stageResults map

                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
						try {					
						if (env.Deployment != '') {						
            deploy adapters: [tomcat8(credentialsId: 'tom', path: '', url: "${env.Deployment}")], contextPath: null, war: '**/*.war'
	   currentStage.resultIsSuccess = true
					        } else {
                                                        echo 'skipping the stage ${env.STAGE_NAME}.............................!'
							currentStage.resultIsSuccess = false
					}	} catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed..................!"
						
						}		
					}
				}
	}
     }
        stage('DAST') {
            steps {
		     
                script {
			def stageResults = [:] // Define the stageResults map

                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    try {
						if (env.DAST_IP != '') {		    
                       arachniScanner checks: '*', format: 'html', url: "${env.DAST_URL}"
			currentStage.resultIsSuccess = true
					        } else {
                                                       echo 'skipping the stage ${env.STAGE_NAME}.............................!'
							currentStage.resultIsSuccess = false
             }       } catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed..................!" 
			    			
                    }
                }
            }
        }	
	}
	stage('FunctionalAutomation_Web') {
           			steps {
					
		   			script {
						def stageResults = [:] // Define the stageResults map

                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
						try {
						if (env.FUNCTIONAL_WEB_GIT_URL != '') {						
							bat  """ git clone ${env.FUNCTIONAL_WEB_GIT_URL}
							cd web_auto/3i-Bank_FalconFramework
							mvn test -Dtest=com.falcon.TestCases.FalconRunnabletest """
							currentStage.resultIsSuccess = true
					        } else {
                                                        echo 'skipping the stage ${env.STAGE_NAME}.............................!'
							currentStage.resultIsSuccess = false
					}	} catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed..................!"
							
						}					
					}
	 		}
	 }
	}
	stage('FunctionalAutomation_Mobile') {
            				steps {
						
		   			script {
						def stageResults = [:] // Define the stageResults map

                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
						try {
						if (env.FUNCTIONAL_MOBILE_GIT_URL != '') {						
							bat """ git clone ${env.FUNCTIONAL_MOBILE_GIT_URL}
							cd mobiletest
							mvn test """
							currentStage.resultIsSuccess = true
					        } else {
                                                        echo 'skipping the stage ${env.STAGE_NAME}.............................!'
							currentStage.resultIsSuccess = false
					}	} catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed..................!"
							
							}
					}
				}
	}
	}
	        stage('Performance') {
            			steps {
					 
		   			script {
						def stageResults = [:] // Define the stageResults map

                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
						try {
						if (env.JMETER_GIT_URL != '') {						
							bat """
                            git clone ${env.JMETER_GIT_URL}
			    dir
                            cd folder
                            C:/ProgramData/Jenkins/.jenkins/workspace/${JOB_NAME}/folder/apache-jmeter-5.4.1/apache-jmeter-5.4.1/bin/jmeter.bat -n -t OnlineShop_1.jmx -l OnlineShop_result.jtl -e -o OnlineShop_%BUILD_NUMBER%.html
            					 """
			 currentStage.resultIsSuccess = true
					        } else {
                                                        echo 'skipping the stage ${env.STAGE_NAME}.............................!'
							currentStage.resultIsSuccess = false
            			}		} catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed..................!"
							
							}
					}
			}
		} 
		}
  	  	   stage('Nexus') {
            steps {
                script {
			 def stageResults = [:] // Define the stageResults map
                    if (stageResults.every { it.value == true }) {
			     // Upload reports to Nexus here for this specific stage
                        bat """
                            curl -v -u admin:admin --upload-file "web-test-report-${BUILD_NUMBER}.zip" "http://10.1.127.197:8081/repository/Flexib-Reports/web-test-report/web-test-report-${BUILD_NUMBER}.zip"
                            curl -v -u admin:admin --upload-file "mobile-test-report-${BUILD_NUMBER}.zip" "http://10.1.127.197:8081/repository/Flexib-Reports/mobile-test-report/mobile-test-report-${BUILD_NUMBER}.zip"
                            curl -v -u admin:admin --upload-file "sast-report-${BUILD_NUMBER}.zip" "http://10.1.127.197:8081/repository/Flexib-Reports/sast-report/sast-report-${BUILD_NUMBER}.zip"
                        """
                    } else {
                echo "At least one stage failed. Skipping report upload to Nexus."
            }
                }
            }
        }   
	}
	post {
   	 always {
		 script {
                if (stageResults.every { it.value == true }) {
        // Clean after build
        mail to: "${env.EMAIL_ID}",
        subject: "Status of pipeline: ${currentBuild.fullDisplayName}",
        body: """Result is ${currentBuild.result}
        
        Please find build reports................................!
        
        Nexus Credentials-(username:admin,password:admin)
        
        SAST REPORTS
        http://10.1.127.197:8081/repository/Flexib-Reports/sast-reports/report-task.txt
        https://sonarcloud.io/dashboard?id=Shoppingcart
        --------------------------------------------------------------------------------------------------------------------------------------
        DAST REPORTS
        http://10.1.127.197:8081/repository/Flexib-Reports/DAST-REPORTS/arachni-report-html-${BUILD_NUMBER}.zip
        ----------------------------------------------------------------------------------------------------------------------------------------
        
        FUNCTIONAL AUTOMATION WEB REPORTS
        http://10.1.127.197:8081/repository/Flexib-Reports/web-test-report/web-test-report-${BUILD_NUMBER}.zip
        ----------------------------------------------------------------------------------------------------------------------------------------
        FUNCTIONAL AUTOMATION MOBILE REPORTS
        http://10.1.127.197:8081/repository/Flexib-Reports/mobile-test-report/mobile-test-report-${BUILD_NUMBER}.zip
        ----------------------------------------------------------------------------------------------------------------------------------------
        
        JMETER REPORTS
        http://10.1.127.197:8081/repository/Flexib-Reports/jmeter-test-report/jmeter/jmeter-test-report-${BUILD_NUMBER}.zip
        
        THE PIPELINE IS SUCCESSFUL AND THIS MAIL IS TO REQUEST FOR QA SIGNOFF AND PROCEED TO UAT OR PROD ENV
        
        For more information, please contact the DEVSECOPS team...........!"""
    }
}
 }
}
}
