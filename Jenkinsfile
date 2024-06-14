namespace = "production"
serviceName = "jobber-review" //kubernets service name for this service
service = "Jobber Reviews" //used when we want to send notification to slack

m1 = System.currentTimeMillis()

def groovyMethods //to load functions.grrovy file content
pipeline{

    agent{
        label 'Jenkins-Agent'
    }
    tools {
        nodejs "NodeJS"
        dockerTool "Docker"
    }

    environment {
        GITHUB_CREDENTIALS =credentials("github")
        IMAGE_NAME ="izuku11" + "/" + "jobber-review"
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
                // create folder pipeline and add the functions.grrovy file from github repo
                sh "[-d pipeline] || mkdir pipeline"
                dir("pipeline"){      //get inside pipeline folder
                    //get code from dev branch
                    git branch: 'dev',credentialsId: 'github',url: 'https://github.com/irshadkhan2019/jenkins-automation'
                    // load file in groovyMethods variable 
                    script {
                        groovyMethods=load "functions.groovy"
                    }
                }

                //fetch repo on jenkins server
                git branch: 'dev',credentialsId: 'github',url: 'https://github.com/irshadkhan2019/Job-app-Review'

                // npmrc 
                sh 'sed -i "s/Removed/$GITHUB_CREDENTIALS_PSW/" "./.npmrc"'
                
                //install service node_modules
                sh 'npm install'
            }
        }

        stage("Build and Push") {
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerpass', usernameVariable: 'dockerUsername')]) {
                    sh 'docker login -u $usernameVariable --password $passwordVariable' //login to dockerhub
                    sh "docker build -t $IMAGE_NAME ."
                    sh "docker tag $IMAGE_NAME $IMAGE_NAME:$IMAGE_TAG"
                    sh "docker tag $IMAGE_NAME $IMAGE_NAME:stable"
                    sh "docker push $IMAGE_NAME:$IMAGE_TAG"
                    sh "docker push $IMAGE_NAME:stable"
                    }
              
                }
         
            }
        }
      // remove build images from jenkins server
        stage("Clean Artifacts") {
            steps {
                sh "docker rmi $IMAGE_NAME:$IMAGE_TAG"
                sh "docker rmi $IMAGE_NAME:stable"
            }
        }

        stage("Create New Pods in kubernetes cluster"){
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', 
                                                            clusterName: 'minikube', contextName: 'minikube', 
                                                            credentialsId: 'jenkins-k8s-token', namespace: 'default',
                                                            serverUrl: 'https://192.168.49.2:8443']]) {
                                
                     script {
                            //get all pod names for servicename 
                            def pods = groovyMethods.findPodsFromName("${namespace}", "${serviceName}")
                                // delete the pod so that it is recreated using lastest image changes  pushed on dockerhub .
                                for (podName in pods) {
                                sh """
                                    kubectl delete -n ${namespace} pod ${podName}
                                    sleep 10s
                                """
                                }
                        }//eoscript         
                }//end of kubecred
            }//eos
        }//eostage

    }//end stages
    
    post {
        // if pipeline succeeds execute this
        success{
            script {
                m2 = System.currentTimeMillis()
                def durTime= groovyMethods.durationTime(m1,m2)
                def author =groovyMethods.readCommitAuthor()
                // slack
                groovyMethods.notifySlack("", "jobber-jenkins", [ //attachment
        				[
        					title: "BUILD SUCCEEDED: ${service} Service with build number ${env.BUILD_NUMBER}",
        					title_link: "${env.BUILD_URL}",
        					color: "good",
        					text: "Created by: ${author}",
        					"mrkdwn_in": ["fields"],
        					fields: [
        						[
        							title: "Duration Time",
        							value: "${durTime}",
        							short: true
        						],
        						[
        							title: "Stage Name",
        							value: "Production",
        							short: true
        						],
        					]
        				]
        		    ]
                )
            }
        }//end of success

        failure {
            script {
                m2 = System.currentTimeMillis()
                def durTime = groovyMethods.durationTime(m1, m2)
                def author = groovyMethods.readCommitAuthor()
                groovyMethods.notifySlack("", "jobber-jenkins", [
                                [
                                    title: "BUILD FAILED: ${service} Service with build number ${env.BUILD_NUMBER}",
                                    title_link: "${env.BUILD_URL}",
                                    color: "error",
                                    text: "Created by: ${author}",
                                    "mrkdwn_in": ["fields"],
                                    fields: [
                                        [
                                            title: "Duration Time",
                                            value: "${durTime}",
                                            short: true
                                        ],
                                        [
                                            title: "Stage Name",
                                            value: "Production",
                                            short: true
                                        ],
                                    ]
                                ]
                        ]
                    )
                }
            }
    }

}
// webhook test