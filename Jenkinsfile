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
        DOCKER_CREDENTIALS =credentials("dockerhub")
        IMAGE_NAME ="izuku11" + "/" + "jobber-review"
        IMAGE_TAG = "stable-${BUILD_NUMBER}"

    }
    stages {
        stage("Cleanup Worspace"){
            steps {
                cleans()
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
    }

}
