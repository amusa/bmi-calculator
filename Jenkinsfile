pipeline {
    //agent any
    agent {
                docker {
                   // image 'node:25-alpine'   
                   image 'node:16.13.1-alpine'
                }
            }
    
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
            steps {
               
                sh 'npm ci'
                sh 'node --version'   
                        
                
                // 2. Execute unit tests and calculate code coverage
                sh 'npm run test -- --coverage --watchAll=false'
                
                // 3. Configure Cobertura plugin with 40% minimum Line Coverage
                // cobertura(
                //     coberturaReportFile: '**/coverage/cobertura-coverage.xml',
                //     lineCoverageTargets: '40',
                //     failUnhealthy: true
                // )

                // recordCoverage(
                //     tools: [[parser: 'COBERTURA', pattern: '**/coverage/cobertura-coverage.xml']],
                //     qualityGates: [[threshold: 40.0, metric: 'LINE', baseline: 'PROJECT', unstable: true]]
                // )
                sh 'ls -r coverage'
                
                recordCoverage(
                    tools: [[parser: 'COBERTURA', pattern: '**/coverage/cobertura-coverage.xml']],
                    qualityGates: [[threshold: 40.0, metric: 'LINE', baseline: 'PROJECT', status: 'FAILURE']]                    
                )
            }
        }
        stage('Build'){
            steps{
                sh 'npm run build'
                zip archive: true, dir: 'build', glob: '', zipFile: 'build.zip'
                archiveArtifacts artifacts: 'build.zip', followSymlinks: false
                stash(includes: 'build.zip', name: 'build')
            }
        }
    
    
        
    }
    
}