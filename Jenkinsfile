pipeline {
   agent {
	label 'Jenkins-Agent'
   }
   tools {
	jdk 'Java17'
	maven 'Maven3'
   }
   stages {
     stage ("Clean Workspace") {
    	steps {
    	   cleanWs()
	}
       }
     stage ("Git checkout") {
	steps {
	   git branch: 'main', url: 'https://github.com/kumarvmadhu/register-app/'
	}
      }
     stage ("Build_App") {
	steps {
	   sh "mvn clean package"
	}
      }
     stage ("Test_App") {
	steps {
	   sh "mvn test"
	}
     }
    stage ("Sonar-Analysis") {
	steps {
	   script {
	     withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn sonar:sonar"
	       }
	   }
        }
     }
  }
}
