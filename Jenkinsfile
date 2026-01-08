pipeline {
    agent any
    
    stages {        
        stage('SonarQube Analysis') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:latest'   
                }
            }
            steps {
                withSonarQubeEnv(installationName: 'SonarQubeServer', credentialsId: 'SonarToken') {  
                    sh 'sonar-scanner -Dsonar.projectVersion=1.0 -Dsonar.projectKey=react-bmi-app -Dsonar.sources=src'                
                }
            }
        }
        stage("Quality Gate") {
            steps {
                waitForQualityGate(abortPipeline: true, credentialsId: 'SonarToken')  
            }
        }
        
        stage("CI Stage"){  
            agent {
                docker {
                   // image 'node:25-alpine'   
                   image 'node:16.13.1-alpine'
                }
            }          
            steps {
                              
                sh 'node --version'   
                sh 'ls -la'
                sh 'rm -f build.zip; rm -rf build'   
                sh 'ls -la'
                sh 'npm ci --cache .npm'
                
                sh 'npm run test -- --coverage --watchAll=false'

                sh 'ls -r coverage'
                
                recordCoverage(
                    tools: [[parser: 'COBERTURA', pattern: '**/coverage/cobertura-coverage.xml']],
                    qualityGates: [[threshold: 40.0, metric: 'LINE', baseline: 'PROJECT', status: 'FAILURE']]                    
                )
            }
        }
        stage('Build'){
            agent { 
                dockerfile { 
                    filename 'Dockerfile.build' 
                    args '-v /root/.npm:/.npm' 
                } 
             }
            steps{
                sh 'npm run build'
                zip archive: true, dir: 'build', glob: '', zipFile: 'build.zip'
                archiveArtifacts artifacts: 'build.zip', followSymlinks: false
                stash(includes: 'build.zip', name: 'build')
            }
        }
        stage('Docker Image') {
            agent{
                docker {
                    image 'docker:27-dind'
                    // args '--privileged'
                    args '-u root'
                }
            }
            steps { 
                unstash 'build' 
                unzip dir: 'build', glob: '', zipFile: 'build.zip' 
                sh 'docker build -f Dockerfile -t ayemi/bmi-calc:1.0 . && docker images' 
                withDockerRegistry([ credentialsId: "dockerhub-cred", url: "" ]) { 
                    sh 'docker push ayemi/bmi-calc:1.0' 
                }
            } 
        }
    
        
    }
    
}