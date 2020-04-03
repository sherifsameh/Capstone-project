pipeline {
	agent any

   	environment {
        DOCKER_IMAGE_NAME = "arsanyatya/capstoneimage"
	}

	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		
		stage('Build Docker Image') {
            		steps {
                		script {
                    			app = docker.build(DOCKER_IMAGE_NAME)
                    			app.inside {
                        			sh 'echo Hello, Nginx!'
                    			}
                		}
            		}

		}

       		 stage('Push Docker Image') {
            		steps {
                		script {
                    			docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        		app.push("${env.BUILD_NUMBER}")
                        		app.push("latest")
                    			}
                		}
            		}
        	}


        	stage('Deploy blue container') {
            		steps {
                          sshagent(['Project']) {
                             sh "scp -o StrictHostKeyChecking=no  blue-controller.yaml ec2-user@35.183.123.18:/home/ec2-user/"
                             script{
                                try{
	                            sh "ssh ec2-user@35.183.123.18 sudo kubectl apply -f ."
	                     }catch(error){
	                            sh "ssh ec2-user@35.183.123.18 sudo kubectl create -f ."
                                          }
                            }
                         }
            	   }
        	}


                stage('Deploy green container') {
                        steps {
                          sshagent(['Project']) {
                             sh "scp -o StrictHostKeyChecking=no  green-controller.yaml ec2-user@35.183.123.18:/home/ec2-user/"
                             script{
                                try{
	                            sh "ssh ec2-user@35.183.123.18 sudo kubectl apply -f ."
	                     }catch(error){
	                            sh "ssh ec2-user@35.183.123.18 sudo kubectl create -f ."
                                          }
                            }
                         }
                        }
                }


		stage('Create the service in the cluster, redirect to blue') {
                        steps {
                          sshagent(['Project']) {
                             sh "scp -o StrictHostKeyChecking=no  blue-service.yaml ec2-user@35.183.123.18:/home/ec2-user/"
                             script{
                                try{
	                            sh "ssh ec2-user@35.183.123.18 sudo kubectl apply -f ."
	                     }catch(error){
	                            sh "ssh ec2-user@35.183.123.18 sudo kubectl create -f ."
                                          }
                            }
                         }
                        }
		}

		stage('Wait user approve') {
            steps {
                input "Ready to redirect traffic to green?"
            }
        }

                stage('Create the service in the cluster, redirect to green') {
                        steps {
                          sshagent(['Project']) {
                             sh "scp -o StrictHostKeyChecking=no  green-service.yaml ec2-user@35.183.123.18:/home/ec2-user/"
                             script{
                                try{
	                            sh "ssh ec2-user@35.183.123.18 sudo kubectl apply -f ."
	                     }catch(error){
	                            sh "ssh ec2-user@35.183.123.18 sudo kubectl create -f ."
                                          }
                            }
                         }
                        }
                }

	}
}
