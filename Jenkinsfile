
 pipeline { 
 	agent any  
 
 	stages { 
		stage('SonarQube Analysis') {
      agent {
        // docker {
        //   image 'sonarsource/sonar-scanner-cli:latest'   
        // }
		// dockerfile{
		// 	filename 'Dockerfile.sonar'
		// }
      }
			stages{
				stage('Scan'){
					steps {
						container('sonar'){
							withSonarQubeEnv(installationName: 'SonarQubeServer', credentialsId: 'SonarToken') {  
								sh 'sonar-scanner -Dsonar.projectVersion=1.0 -Dsonar.projectKey=react-bmi-app -Dsonar.sources=src'                
							}
						}
      				}
				}
				stage("Quality Gate") {
					steps {
						waitForQualityGate(abortPipeline: true, credentialsId: 'SonarToken')  
					}
    		}

			}
      
    }
    
  	stage('CI') { 
 			agent { 
 				dockerfile { 
 					filename 'Dockerfile.build' 
 					args '-v /root/.npm:/.npm' 
 				} 
 			} 
 
			stages { 
				stage('NPM') { 
					steps { 
						sh 'ls -a; node --version' 
						sh 'rm -f build.zip; rm -rf build' 
						sh 'npm ci --cache .npm' 
					} 
 				} 
 				stage('Build') { 
					steps { 
						sh 'npm run build' 
						zip archive: true, dir: 'build', glob: '', zipFile: 'build.zip' 
						archiveArtifacts artifacts: 'build.zip', followSymlinks: false 
						stash(includes: 'build.zip', name: 'build') 
 					} 
 				} 
 
 			} 
 		} 
 		stage('Docker Image') {
			steps { 
				unstash 'build' 
				unzip dir: 'build', glob: '', zipFile: 'build.zip' 
				sh 'docker build -f Dockerfile -t ayemi/bmi-calc:1.0 . && docker images' 
			
				withDockerRegistry([ credentialsId: "dockerhub-cred", url: "" ]) { 
					sh 'docker push ayemi/bmi-calc:1.0' 
				} 
			} 
		} 
		stage('Trivy Image Scan'){
			agent {
				dockerfile {
					filename 'Dockerfile.docker'
					args '--entrypoint='
				}
			}
			steps {
				sh "trivy help && trivy --cache-dir /tmp/trivy/ image -f json -o results.json 'ayemi/bmi-calc:1.0'"
				recordIssues(tools: [trivy(pattern: 'results.json')])
			}
		}
		stage('Minikube Deployment'){
			agent{
				label 'kubernetes'
			}
			steps{
				sh 'kubectl apply -f react-deployment.yaml'
			}
		}
		
	}
	
}