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
                    try {
                // Get some code from a GitHub repository
						if (env.SAST_GIT_URL != '') {				
                git "${env.SAST_GIT_URL}"
                bat "git clone https://github.com/harishpallapu/sonarqube_scannar_windows.git"
                bat "./sonarqube_scannar_windows/sonar-scanner-4.6.2.2472-windows/bin/sonar-scanner.bat"
					        } else {
                                                        echo "skipping the stage ${env.STAGE_NAME}.............................!"				
                   } } catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed...............!"
						}                
                // Run Maven on a Unix agent.
                //sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        } 
        }
       stage('SCA') {
    steps {
        script {
            try {
                // Download the DependencyCheck release zip using curl
                bat 'curl -o dependency-check-6.1.5-release.zip https://github.com/jeremylong/DependencyCheck/releases/download/v6.1.5/dependency-check-6.1.5-release.zip'
                
                // Extract the downloaded zip using PowerShell
                 bat "PowerShell Expand-Archive -Path \"${WORKSPACE}\\dependency-check-6.1.5-release.zip\" -DestinationPath \"${WORKSPACE}\""
		// bat "PowerShell Expand-Archive -Path \"${WORKSPACE}\\dependency-check-6.2.2-release.zip\" \"${WORKSPACE}\""

                // Run DependencyCheck
                bat "\"${WORKSPACE}\\dependency-check-6.1.5-release\\dependency-check\\bin\\dependency-check.bat\" --noupdate --project \"TeachersFCU\" --scan \"Shoppingcart/lib/\" --format HTML --out \"${WORKSPACE}\""

       		     } catch (err) {
                echo err.getMessage()
                unstable(message: "${STAGE_NAME} is unstable")
                echo "Error detected, ${env.STAGE_NAME} failed..................!"
    	        }
     	   }
   	 }
	}

	stage('build') {
            steps {
				script {
						try {
						if (env.BUILD_GIT_URL != '') {
                git "${env.BUILD_GIT_URL}"
							bat 'mvn clean package'
					        } else {
                                                        echo 'skipping the stage ${env.STAGE_NAME}.............................!'
                                                        }							
						} catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed..................!"
						}
				}							
            }
	 }
     stage("deploy") {
         steps {
					script {
						try {					
						if (env.Deployment != '') {						
            deploy adapters: [tomcat8(credentialsId: 'tom', path: '', url: "${env.Deployment}")], contextPath: null, war: '**/*.war'
					        } else {
                                                        echo 'skipping the stage ${env.STAGE_NAME}.............................!'	    
					}	} catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed..................!"
						}		
					}
				}
	}
        stage('DAST') {
            steps {
                script {
                    try {
						if (env.DAST_IP != '') {		    
                       arachniScanner checks: '*', format: 'html', url: "${env.DAST_URL}"
					        } else {
                                                       echo 'skipping the stage ${env.STAGE_NAME}.............................!'                           
             }       } catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed..................!"                       
                    }
                }
            }
        }	
	
	stage('FunctionalAutomation_Web') {
           			steps {
		   			script {
						try {
						if (env.FUNCTIONAL_WEB_GIT_URL != '') {						
							bat  """ git clone ${env.FUNCTIONAL_WEB_GIT_URL}
							cd web_auto/3i-Bank_FalconFramework
							mvn test -Dtest=com.falcon.TestCases.FalconRunnabletest """
					        } else {
                                                        echo 'skipping the stage ${env.STAGE_NAME}.............................!'														
					}	} catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed..................!"
						}					
					}
	 		}
	 }
	stage('FunctionalAutomation_Mobile') {
            				steps {
		   			script {
						try {
						if (env.FUNCTIONAL_MOBILE_GIT_URL != '') {						
							bat """ git clone ${env.FUNCTIONAL_MOBILE_GIT_URL}
							cd mobiletest
							mvn test """
					        } else {
                                                        echo 'skipping the stage ${env.STAGE_NAME}.............................!'							
					}	} catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed..................!"
							}
					}
				}
	}
	        stage('Performance') {
            			steps {
		   			script {
						try {
						if (env.JMETER_GIT_URL != '') {						
							bat """
                            git clone ${env.JMETER_GIT_URL}
			    dir
                            cd folder
                            C:/ProgramData/Jenkins/.jenkins/workspace/${JOB_NAME}/folder/apache-jmeter-5.4.1/apache-jmeter-5.4.1/bin/jmeter.bat -n -t OnlineShop_1.jmx -l OnlineShop_result.jtl -e -o OnlineShop_%BUILD_NUMBER%.html
            					 """
					        } else {
                                                        echo 'skipping the stage ${env.STAGE_NAME}.............................!'						 
            			}		} catch (err) {
							echo err.getMessage()
							unstable(message: "${STAGE_NAME} is unstable")
							echo "Error detected, ${env.STAGE_NAME} failed..................!"
							}
					}
			}
		} 			
        stage('Nexus') {
            steps {
                
              
	       bat 'tar -c -f web-test-report-%BUILD_NUMBER%.zip web_auto/3i-Bank_FalconFramework/Report/* '
		// bat "tar -a -c -f \"${WORKSPACE}\\web-test-report-12.zip\" \"${WORKSPACE}\\web_auto\\3i-Bank_FalconFramework\\Report\\*\""
	       bat 'tar -c -f mobile-test-report-%BUILD_NUMBER%.zip mobiletest/target/surefire-reports/*'
	       bat 'tar -c -f jmeter-test-report-%BUILD_NUMBER%.zip folder/OnlineShop_%BUILD_NUMBER%.html/*'


             //  bat '''       
              // curl -T "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\%JOB_NAME%\\Shoppingcart\\.scannerwork\\report-task.txt" -u admin:flexib -v http://10.1.127.197:8081/#browse/browse:Flexib-Reports:sast-reports
	      // curl -T "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\%JOB_NAME%\\arachni-report-html.zip" -u admin:flexib -v http://10.1.127.197:8081/#browse/browse:Flexib-Reports:DAST-REPORTS
              // curl -T "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\%JOB_NAME%\\web-test-report-%BUILD_NUMBER%.zip" -u admin:flexib -v http://10.1.127.197:8081/#browse/browse:Flexib-Reports:web-test-report 
              // curl -T "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\%JOB_NAME%\\mobile-test-report-%BUILD_NUMBER%.zip" -u admin:flexib -v http://10.1.127.197:8081/#browse/browse:Flexib-Reports:mobile-test-report      
              // curl -T "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\%JOB_NAME%\\jmeter-test-report-%BUILD_NUMBER%.zip" -u admin:flexib -v http://10.1.127.197:8081/#browse/browse:Flexib-Reports:jmeter
               '''
	     
  		def buildNumber = env.BUILD_NUMBER
		bat """
		curl -T \"web-test-report-${buildNumber}.zip\" -u admin:flexib -v \"http://10.1.127.197:8081/repository/Flexib-Reports/web-test-report/web-test-report-${buildNumber}.zip\"
		"""


                           }
        }              
    }		
 	post {
        // Clean after build
        always {
	mail to: "${env.EMAIL_ID}",
        subject: "Status of pipeline: ${currentBuild.fullDisplayName}",
        body: """Result is ${currentBuild.result}
        
		Please find build reports................................!
		
		Nexus Credentials-(username:admin,password:flexib)
		
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
		
		
		
		THE PIPELINE IS SUCCESSFULL AND THIS MAIL IS TO REQUEST FOR QA SIGNOFF AND PROCEED TO UAT 0R PROD ENV
		
		For more information please contact DEVSECOPS team...........!"""

}
}
}
