
 pipeline { 
 	agent none
 
 	stages { 

    
  	stage('CI') { 
 			agent { 
				label 'node-build-agent'
 			} 
 
			stages { 
				stage('NPM') { 
					steps { 
						container('node') {
							sh 'ls -a; node --version' 
							sh 'rm -f build.zip; rm -rf build' 
							// sh 'npm ci --cache /tmp/.npm-cache' 
							sh 'npm install' 
						}
					} 
 				} 
 				stage('Build') { 
					steps { 
						container('node') {
							sh 'npm run build' 
							zip archive: true, dir: 'build', glob: '', zipFile: 'build.zip' 
							archiveArtifacts artifacts: 'build.zip', followSymlinks: false 
							stash(includes: 'build.zip', name: 'build') 
						}
 					} 
 				} 
 
 			} 
 		} 
 		stage('Docker Image') {
			agent { 
				label 'docker-build'
 			} 
			steps { 
				container('docker'){

					sh '''
						echo "Current Shell: $0"
						which bash || echo "bash not found"
						which docker || echo "docker not found"
						// ls -l /bin/bash /usr/bin/bash || true
					'''


					unstash 'build' 
					unzip dir: 'build', glob: '', zipFile: 'build.zip' 
					
					sh '''#!/bin/sh
						# Wait for docker daemon to be ready
						until docker info >/dev/null 2>&1; do
							echo "Waiting for Docker daemon..."
							sleep 2
						done
						docker build -f Dockerfile -t ayemi/bmi-calc:1.0 . && docker images
					'''
					withDockerRegistry([ credentialsId: "dockerhub-cred", url: "" ]) { 
					 	sh 'docker push ayemi/bmi-calc:1.0' 					
					} 
				}
			} 
		} 
		stage('Minikube Deployment'){
			agent{
				kubernetes{
					inheritFrom 'k8s-agent'
				}
			}
			steps{
				container('kubectl'){
					sh 'kubectl apply -f react-deployment.yaml -n bmi-app'
				}
			}
		}
		
	}
	
}