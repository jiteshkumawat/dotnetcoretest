pipeline{
	agent any

environment
{
    scannerHome = tool name: 'sonar_scanner_dotnet', type: 'hudson.plugins.sonar.MsBuildSQRunnerInstallation'   
    dotnet = 'C:\\Program Files\\dotnet\\dotnet.exe'
}
options
   {
      timeout(time: 1, unit: 'HOURS')
      
      // Discard old builds after 5 days or 5 builds count.
      buildDiscarder(logRotator(daysToKeepStr: '5', numToKeepStr: '5'))
	  
	  //To avoid concurrent builds to avoid multiple checkouts
	  disableConcurrentBuilds()
   }
     
stages
{
	stage ('checkout')
    {
		steps
		{
			echo  " ********** Clone starts ******************"
		    checkout scm	 
		}
    }
    stage ('nuget')
    {
		steps
		{
			bat "dotnet restore"	 
		}
    }
	stage ('Start sonarqube analysis')
	{
		steps
		{
			withSonarQubeEnv('jitesh_assignment')
			{
				bat "dotnet \"${scannerHome}/SonarScanner.MSBuild.dll\" begin /k:$JOB_NAME /n:$JOB_NAME /v:1.0 "    
			}
		}
	}
	stage ('build')
	{
		steps
		{
			bat "dotnet build -c Release -o WebApplication4/app/build"
		}	
	}
	stage ('SonarQube Analysis end')
	{	
		steps
		{
		    withSonarQubeEnv('jitesh_assignment')
			{
				bat "dotnet \"${scannerHome}/SonarScanner.MSBuild.dll\" end"
			}
		}
	}
	
	stage ('Release Artifacts')
	{
	    steps
	    {
	        bat "dotnet publish -c Release -o WebApplication4/app/publish"
	    }
	}
	stage ('Docker Image')
	{
		steps
		{
		    bat "docker build --no-cache -t jiteshkum/nagp_assignment:${BUILD_NUMBER} ."
		}
	}
	stage ('Push to DTR')
	{
		steps
		{
			bat "docker login --username jiteshkum --password Passw0rd@1801"
			bat "docker push jiteshkum/nagp_assignment:${BUILD_NUMBER}"
		}
	}
	
	stage ('Stop Running container')
	{
	    steps
	    {
		    bat "echo (docker ps --format \"{{.ID}}\")"
		    bat "docker ps --format \"{{.ID}}\""
		    bat "echo (docker ps --format \"{{.Names}}\")"		    
		    bat "docker ps --format \"{{.Names}}\""
	    }
	}
	
	stage ('Docker deployment')
	{
	    steps
	    {
	       bat "docker run --name nagp_assignment -d -p 5000:80 jiteshkum/nagp_assignment:${BUILD_NUMBER}"
	    }
	}
}

 post {
        always 
		{
			echo "*********** Executing post tasks like Email notifications *****************"
        }
    }
}
