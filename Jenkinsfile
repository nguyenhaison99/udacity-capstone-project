pipeline {
	agent any
	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'udacity-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t nguyenson99/udacity-capstone .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'udacity-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push nguyenson99/udacity-capstone
					'''
				}
			}
		}

		stage('Set Current kubectl Context') {
			steps {
				withAWS(region:'us-east-1', credentials:'udacity-capstone') {
                    withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) {
                    // Your stuff here
                    sh '''
                        sudo chown -R $USER /var/lib/jenkins/.kube/kubeconfig
						kubectl config use-context arn:aws:eks:us-east-1:125745568001:cluster/udacitycluster
					'''
                    }	
				}
			}
		}

		stage('Deploy Blue Container') {
			steps {
				withAWS(region:'us-east-1', credentials:'udacity-capstone') {
                    withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) {
                    // Your stuff here
                    sh '''
						kubectl apply -f ./kubernetes-resources/blue-replication-controller.yml --context udacitycluster
					'''
                    }	
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-east-1', credentials:'aws_credentials') {
                    withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) { 
					sh '''
						kubectl apply -f ./kubernetes-resources/green-replication-controller.yml --context udacitycluster
					'''
				    }
                }
			}
		}

		stage('Create Service Pointing to Blue Replication Controller') {
			steps {
				withAWS(region:'us-east-1', credentials:'aws_credentials') {
                    withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) {
					sh '''
						kubectl apply -f ./kubernetes-resources/blue-service.yml --context udacitycapstonecluster
					'''
                    }
                }
			}
		}

		stage('Approval for Redirection') {
            steps {
                input "Ready to redirect traffic to green replication controller?"
            }
        }

		stage('Create Service Pointing to Green Replication Controller') {
			steps {
				withAWS(region:'us-east-1', credentials:'aws_credentials') {
					withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) {
                    sh '''
						kubectl apply -f ./kubernetes-resources/green-service.yml --context udacitycapstonecluster
					'''
                    }
                }
			}
		}

	}
}
