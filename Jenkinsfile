pipeline {
    agent any
		
	environment {
		scannerHome = tool name: 'sonar_scanner_dotnet'
		username = 'deepmalamittal'
   	}	
   
	options {
        timestamps()
		timeout(time: 1, unit: 'HOURS') 	
    }
    
    stages {   
    	stage ("Nuget restore") {
            steps {	    
                echo "Restoring nuget packages"
                sh "dotnet restore"
            }
        }
		
		stage('Start sonarqube analysis'){
            when {
                branch "master"
            }
            steps {
				   withSonarQubeEnv('Test_Sonar') {
                   sh "dotnet ${scannerHome}/SonarScanner.MSBuild.dll begin /k:sonar-${userName} /n:sonar-${userName} /v:1.0"
                }
            }
        }

        stage('Code build') {
            steps {		  
				  echo "Cleaning and building the code.."
                  sh "dotnet clean"		  
                  sh "dotnet build --configuration Release"     
            }
        }
		
		stage('Test case Execution') {
             when {
                branch "master"
            }
			
			steps {			  
				  echo "Execute test cases."
                  sh "dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=TestFileReport.xml"      
            }
        }
		stage('Stop sonarqube analysis'){
             when {
                branch "master"
            }
            
			steps {
				   echo "Stopping sonarqube analysis"
                   withSonarQubeEnv('Test_Sonar') {
                   sh "dotnet ${scannerHome}\\SonarScanner.MSBuild.dll end"
                   }
            }
        }
        stage ("Release artifact") {
            when {
                branch "develop"
            }
            steps {
                echo "Releasing the artifact"
                sh "dotnet publish --configuration Release"
            }
        }

        stage('Kubernetes Deployment') {
		  steps{
             echo "Initiating Kubernetes deployment"
		     sh "kubectl apply -f deployment.yaml"
		    }
		}
   	}
}
