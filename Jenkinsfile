pipeline{
    agent any
    environment{
		registryName = "CoitFrontend1"
		registryUrl = "coitfrontend1.azurecr.io"
		registryCredential = "ACR"
		COMPOSE_PROJECT_NAME = "mycustomsolution"

		mavenHome = tool 'mymaven'
		dockerHome = tool 'mydocker'
		gitHome = tool 'myGit'
		PATH = "$dockerHome/bin:$mavenHome/bin:$gitHome/bin:$PATH"
	}
	stages{
		stage('Checkout'){
			steps{
				sh 'mvn --version'
				sh 'docker --version'
				echo "Build"
				echo "PATH - $PATH"
				echo "BUILD_NUMBER - $env.BUILD_NUMBER"
				echo "BUILD_ID - $env.BUILD_ID"
				echo "BUILD_TAG - $env.BUILD_TAG"
				echo "JOB_NAME - $env.JOB_NAME"
				//checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [],
				// gitTool: 'Default', userRemoteConfigs: [[url: 'https://github.com/kollidatta/Coit-Jenkins']]]) 
			}	
		}
        stage('Build '){
            steps{
                dir('./coit-frontend'){
				//echo "path- $PATH"
				script{
				def FRONTENDDOCKER = 'Dockerfile-multistage'
				DockerFrontend = docker.build("dheerajlearningdocker/frontend:${env.BUILD_TAG}","-f ${FRONTENDDOCKER} .")
				//sh('docker build -t kollidatta/coitfrontend:v1 -f Dockerfile-multistage .')
				
				}
				} 
            }
        }
		stage('Push Frontend to ACR'){
			steps{
				script{
					docker.withRegistry("http://${registryUrl}",registryCredential){
						DockerFrontend.push();
						DockerFrontend.push('latest');
					}
				}

			}
		}
		stage('Build backend2'){
            steps{
                dir('./coit-backend2'){
				echo "path- $PATH"
				script{
				DockerBackend2 = docker.build("dheerajlearningdocker/backend2:${env.BUILD_TAG}")
				//sh('docker build -t kollidatta/coitbackend2:v1 -f ./coit-backend2/Dockerfile .')
				}
				}
                
            }
        }
		stage('Push backend2'){
			steps{
				script{
					//docker.withRegistry('','dockerhub'){
					docker.withRegistry("http://${registryUrl}",registryCredential){
						DockerBackend2.push();
						DockerBackend2.push('latest');
					}
				}

			}
		}
		stage('Build backend1'){
            steps{
                dir('./coit-backend1'){
					script{
					def BACKENDDCKER = 'Dockerfile-multistage'
					DockerBackend1 = docker.build("dheerajlearningdocker/backend1:${env.BUILD_TAG}","-f ${BACKENDDCKER} .")
					//sh('docker build -t kollidatta/coitbackend2:v1 -f ./coit-backend2/Dockerfile .')
					}
				}
                
            }
        }
		stage('Push backend1'){
			steps{
				script{
					docker.withRegistry("http://${registryUrl}",registryCredential){
						DockerBackend1.push();
						DockerBackend1.push('latest');
					}
				}

			}
		}
		stage('Deploy to K8s'){
			steps{
				dir('./resource-manifests'){
					script{
						kubernetesDeploy(
							configs:"service-coit-frontend-lb.yaml",
							kubeconfigId:"K8S",
							enableConfigSubstitution:true
						)
						}
					}		
				}
			}
		stage('Deploy backend2 to K8s'){
			steps{
				dir('./resource-manifests'){
					script{
						kubernetesDeploy(
							configs:"coit-frontend-deployment.yaml",
							kubeconfigId:"K8S",
							enableConfigSubstitution:true
						)
						}
					}		
				}
			}
    }
	post {
			always{
				echo "Im awesome. I run always"
			}
			success{
				echo 'I run when you are successful'
			}
			failure{
				echo 'I run when you fail'
			}
		}
	

}