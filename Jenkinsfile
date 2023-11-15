pipeline {
   agent {
	label 'Jenkins-Agent'
   }
   tools {
	jdk 'Java17'
	maven 'Maven3'
   }
   environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "kumarvmadhu"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	   JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
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
    	     withSonarQubeEnv(credentialsId: 'sonar-token') { 
                        sh "mvn sonar:sonar"
    	       }
   	   }
        }
     }
   stage ("Quality-gate") {
      steps { 
	   waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
       }
     }
   stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                    docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
            }
	}
     }
     stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image kumarvmadhu/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }
     stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }
      stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-3-101-17-185.us-west-1.compute.amazonaws.com:8080/job/gitops-cd-job/buildWithParameters?token=gitops-token'"
                }
            }
       }
    }
}
