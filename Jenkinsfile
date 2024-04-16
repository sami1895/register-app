
pipeline {
    agent any
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "application-java"
	DOCKERHUB_CREDENTIALS = credentials('dockerhub')
	RELEASE = "1.0.0"
	DOCKER_USER = "imas10"
        DOCKER_PASS = 'dockerhub'
	IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
	IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")    
    }
	
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        

    }
}
      
