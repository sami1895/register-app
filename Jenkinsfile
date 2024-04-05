
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

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/sami1895/register-app.git'
                }
        }

      stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }  
       stage("SonarQube Analysis"){
           steps {
	       script {
		    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                    sh "mvn sonar:sonar"
		    }
	       }	
           }
       }
       stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }	
            }

       }
       stage('Docker Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }    

        stage("Build & Push Docker Image") {
    steps {
        script {
            sh "ansible-playbook playbook.yml --extra-vars 'IMAGE_NAME=my_image_name IMAGE_TAG=my_image_tag'"
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
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://192.168.50.10:8080//job/gitops-register-app/buildWithParameters?token=gitops-token'"
                }
            }
       }
       stage("slack") {
           steps {
	          slackSend channel: '#jenkins',
                  color: 'good',
                  failOnError: true,
                  message: "Successful completion of ${env.JOB_NAME} (<${env.BUILD_URL}|Open>)",
                  tokenCredentialId: 'slack'

	    }
        }    

    }
}
      
