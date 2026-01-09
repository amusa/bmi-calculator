
 pipeline { 
 agent any 
 
 stages { 
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
 } 
 }