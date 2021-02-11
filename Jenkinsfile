pipeline {
    agent any
    tools {
        maven "Maven" 
	  
	    
    }
   stages {
   	/*stage("SonarQube Analysis"){
        	steps {
                	withSonarQubeEnv("Sonarqube") {
                    		sh "mvn -f pom.xml sonar:sonar -Dsonar.sources=src/"
                    		script {
		    			LAST_STARTED = env.STAGE_NAME
					container_Up = false
                    			timeout(time: 1, unit: "HOURS") { 
                        			sh "curl -u admin:admin -X GET -H \"Accept: application/json\" http://104.248.169.167:9000/api/qualitygates/project_status?projectKey=com.mycompany:wednesdayazuredemo > status.json"
                        			def json = readJSON file:"status.json"
                        			echo "${json.projectStatus}"
                        			if ("${json.projectStatus.status}" != "OK") {
                            				currentBuild.result = "FAILURE"
                           				error("Pipeline aborted due to quality gate failure.")
                           			}
                        		}     
                    		}
                	}
                }
	}*/
      
	 stage('SonarQube Analysis'){
            steps {
                sh "mvn -f pom.xml -Dsonar.login=4ad13676dc37c38930156dcff0a40c7e8f4647fd -Dsonar.host.url=http://104.248.169.167:9000 sonar:sonar "
            }
        }
	 stage('Build') {
            steps {
				 withCredentials([file(credentialsId: 'settings', variable: 'settings')]){
				sh "mvn -f pom.xml -s $settings clean install -DskipTests "     
				}
            		
                  }    
        } 
        stage ('Munit Test'){
        	steps {
				withCredentials([file(credentialsId: 'settings', variable: 'settings')]){
					sh "mvn -f pom.xml -s $settings test -Dkey=mymulesoft -Dhttp.port=8086"
					publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/site/munit/coverage', reportFiles: 'summary.html', reportName: 'Munit coverage Report', reportTitles: ''])
				}
        		   
        	      }    
        }
        stage('Functional Testing'){
        	steps {
        			withCredentials([file(credentialsId: 'settings', variable: 'settings')]){
						sh "mvn -f pom.xml -s $settings test -Dtestfile=src/test/javarunner.TestRunner.java -Dkey=mymulesoft -Dhttp.port=8086"
					}
             	  }
            }
        stage('Generate Reports') {
      		steps {
        		    cucumber(failedFeaturesNumber: -1, failedScenariosNumber: -1, failedStepsNumber: -1, fileIncludePattern: 'cucumber.json', jsonReportDirectory: 'results', pendingStepsNumber: -1, skippedStepsNumber: -1, sortingMethod: 'ALPHABETICAL', undefinedStepsNumber: -1)
                  }
            }
        stage('Archetype'){
        	steps {
                    withCredentials([file(credentialsId: 'settings', variable: 'settings')]){
						sh "mvn -f pom.xml -s $settings archetype:create-from-project -Dkey=mymulesoft"
                    sh "mvn -f target/generated-sources/archetype/pom.xml -s $settings clean install -Dkey=mymulesoft"
					}
                  } 
        	}    
        stage('Deploy to Cloudhub'){
        	steps {
				 withCredentials([file(credentialsId: 'settings', variable: 'settings')]){
        	    	sh "mvn -f pom.xml -s $settings package deploy -DskipTests -Dusername=joji1 -Dpassword=Canadavisa25@ -DapplicationName=wednesdayazuredemo-poc -Denvironment=Sandbox -DmuleDeploy -Dcloudhub.region=us-east-2 -Danypoint.platform.client_id=fda777bd3e3b4fcb93aff995fea2043d -Danypoint.platform.client_secret=4193AA1986054C548Bf757fd1B7F6f18 -Dkey=mymulesoft"

				 }
             	  }
            }
	   
    
}
}
