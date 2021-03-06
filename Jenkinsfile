pipeline {
    environment {
        ClusterName = 'capstone-udacity'
        awsRegion = 'us-west-2'
        registry = "udacity1project/capstone"
        registryCredential = 'dockerhub'
        version = 'latest'
        imageName = "capstone"
    }
    agent any
    stages {
        stage('Create EKS')  {
            //when {branch 'test'}
            steps {
                withAWS(credentials: 'aws-static', region: awsRegion) {
                    sh 'echo -e "Please wait for about 15 minutes to finish EKS creation"'
                    sh 'eksctl create cluster --name ${ClusterName} --version 1.13 --nodegroup-name standard-workers --node-type t2.small --nodes 2 --nodes-min 1 --nodes-max 3 --node-ami auto'
                }
            }
        }
        stage('Lint') {
            steps {
                sh 'tidy -q -e **/*.html'
                sh '''docker run --rm -i hadolint/hadolint < Dockerfile'''

            }    
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$version"
                }
            }
        }
        stage('Push Image to Docker Hub') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                    dockerImage.push()
                    }
                }
            }
        }
        stage('Set kubectl EKS context')  {
            steps {
                withAWS(credentials: 'aws-static', region: awsRegion) {
                    sh 'aws eks update-kubeconfig --name ${ClusterName}'
                    sh 'sudo kubectl config use-context arn:aws:eks:us-west-2:543805437419:cluster/${ClusterName}'
                }
            }
        }
        stage('Deploy blue Container')  {
            steps {
                withAWS(credentials: 'aws-static', region: awsRegion) {
                    sh 'kubectl apply -f blue-green/deploy-blue.yaml'
                }
            }
        }
        stage('Deploy green Container')  {
            steps {
                withAWS(credentials: 'aws-static', region: awsRegion) {
                    sh 'kubectl apply -f blue-green/deploy-green.yaml'
                }
            }
        }
		stage('Create the blue service') {
			steps {
				withAWS(credentials: 'aws-static', region: awsRegion) {
					sh 'kubectl apply -f blue-green/blue-service.yaml'
                    sleep 10 //to have time getting service
                    sh 'kubectl get service capstone'
				}
			}
		}

		stage('User approval to deploy the green service') {
            steps {
                input "Redirect service to green?"
            }
        }

		stage('Create the green service') {
			steps {
				withAWS(credentials: 'aws-static', region: awsRegion) {
					sh 'kubectl apply -f blue-green/green-service.yaml'
                    sleep 10 //to have time getting service
                    sh 'kubectl get service capstone'
				}
			}
		}

    }
}