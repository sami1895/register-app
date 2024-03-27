pipeline {
    agent any
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "register-app-pipeline"
	DOCKERHUB_CREDENTIALS = credentials('dockerhub')
	RELEASE = "1.0.0"
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
        stage('Build & push Dockerfile') {
            steps {
                sh """
                ansible-playbook playbook.yml
                """
            }
        } 
	stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image imas10/application-java:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table --timeout 15m')         
           }
       }    
    }
       stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://192.168.50.10:8080//job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
       }
    } 
}
	    
}
      
