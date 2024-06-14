namespace = "production"
serviceName = "jobber-review" //kubernets service name for this service
service = "Jobber Reviews" //used when we want to send notification to slack

pipeline{
    agent{
        label 'Jenkins-Agent'
    }
    tools {
        nodejs "NodeJS"
        dockerTool "Docker"
    }

    environment {
        DOCKER_CREDENTIALS =credentials("dockerhub") //stores in DOCKERHUB_CREDENTIAL_USR ,DOCKERHUB_CREDENTIALS_PSW
        IMAGE_NAME ="izuku11" + "/" + "jobber-review-jenkins"
        IMAGE_TAG = "stable-${BUILD_NUMBER}"

    }
    stages {
        stage("Cleanup Worspace"){
            steps {
                cleanWs()
            }
        }


        stage("Prepare Environment"){
            steps {
                //fetch repo on jenkins server
                git branch: 'main',credentialsId: 'github',url: 'https://github.com/irshadkhan2019/Job-app-Review'
                //install service node_modules
                sh 'npm install'
            }
        }
        stage("Build and Push") {
            steps {
                sh 'docker login -u $DOCKERHUB_CREDENTIAL_USR --password $DOCKERHUB_CREDENTIALS_PSW' //login to dockerhub
                sh "docker build -t $IMAGE_NAME ."
                sh "docker tag $IMAGE_NAME $IMAGE_NAME:$IMAGE_TAG"
                sh "docker tag $IMAGE_NAME $IMAGE_NAME:stable"
                sh "docker push $IMAGE_NAME:$IMAGE_TAG"
                sh "docker push $IMAGE_NAME:stable"
            }
        }
      // remove build images from jenkins server
        stage("Clean Artifacts") {
            steps {
                sh "docker rmi $IMAGE_NAME:$IMAGE_TAG"
                sh "docker rmi $IMAGE_NAME:stable"
            }
        }

    }//end stages

}
