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
                		kubernetesDeploy(
                    			kubeconfigId: 'kubeconfig',
                    			configs: 'blue-controller.yaml',
                    			enableConfigSubstitution: true
                		)
            		}
        	}


                stage('Deploy green container') {
                        steps {
                                kubernetesDeploy(
                                        kubeconfigId: 'kubeconfig',
                                        configs: 'green-controller.yaml',
                                        enableConfigSubstitution: true
                                )
                        }
                }


		stage('Create the service in the cluster, redirect to blue') {
                        steps {
                                kubernetesDeploy(
                                        kubeconfigId: 'kubeconfig',
                                        configs: 'blue-service.yaml',
                                        enableConfigSubstitution: true
                                )
                        }
		}

		stage('Wait user approve') {
            steps {
                input "Ready to redirect traffic to green?"
            }
        }

                stage('Create the service in the cluster, redirect to green') {
                        steps {
                                kubernetesDeploy(
                                        kubeconfigId: 'kubeconfig',
                                        configs: 'green-service.yaml',
                                        enableConfigSubstitution: true
                                )
                        }
                }

	}
}
