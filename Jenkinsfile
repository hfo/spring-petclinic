node {
    def mvnHome
    mvnHome = tool 'mvn'
    env.PATH = "${mvnHome}/bin:${env.PATH}"
    
    stage('GIT') { 
        git url:'https://github.com/hfo/spring-petclinic.git'
    }
    
    stage('Maven-Test'){
        //sh './mvnw clean test'
    }
   
    stage('Maven-Build') {
        sh './mvnw clean install -Dmaven.test.skip=true'
    }
    
    stage('SonarQube analysis') {
        sh "./mvnw sonar:sonar -Dsonar.host.url=http://172.16.20.157:9000"
        //${mvnHome}/bin/mvn
    }
    
    stage('Nexus Upload Artifact'){
        build job: 'nexus_uploader_job'
    }
    
    
    stage('Dockerize'){
        
        sh 'rm -f Dockerfile'
        sh 'rm -f petclinic-1.0.0.jar'
        
        sh'wget http://172.16.20.157:8081/repository/Jenkins-Repo/Dockerfile'
        sh'wget http://172.16.20.157:8081/repository/Jenkins-Repo/de/proficom/cdp/petclinic/1.0.0/petclinic-1.0.0.jar'
        
        sh 'docker build -t petclinic_alpine .'
   }
    
   stage('Nexus upload Docker image'){
       sh 'docker tag petclinic_alpine 172.16.20.157:8082/petclinic_alpine'
       sh 'docker push 172.16.20.157:8082/petclinic_alpine'
   }
    
   //checkpoint 'before Create VM & Deploy App'
    
   stage('Create VM & Deploy App'){
       
       hook = registerWebhook()
       
       vmName = 'Debian-Runner-'${BUILD_NUMBER}'-TJ'
       
       ip = '172.16.20.116'
       
       build job: 'oo_create_runner_vm_from_template', parameters: [[$class: 'StringParameterValue', name: 'hook_url', value: hook.getURL()],[$class: 'StringParameterValue', name: 'vmName', value: vmName],[$class: 'StringParameterValue', name: 'IP_new', value: ip]]
       
       //sh 'docker pull 172.16.20.157:8082/petclinic_alpine' 
       //sh 'docker run -d -p 4100:8080 petclinic_alpine java -jar /usr/src/petclinic/petclinic-1.0.0.jar'
 
       echo "Waiting for POST to ${hook.getURL()}"
 
       data = waitForWebhook hook
       
       echo "Webhook called with data: ${data}, VM created and started successfully"
 
       sh 'ssh -o StrictHostKeyChecking=no administrator@172.16.20.116 "docker run --rm -d -p 4000:8080 172.16.20.157:8082/petclinic_alpine java -jar /usr/src/petclinic/petclinic-1.0.0.jar"'
       
       //alternatively deploy via ssh ==> does not close the pipeline atm
       
       //sh 'ssh administrator@172.16.20.93 "rm -f petclinic-1.0.0.jar; wget http://172.16.20.92:8081/repository/Jenkins-Repo/de/proficom/cdp/petclinic/1.0.0/petclinic-1.0.0.jar; ls"'
       //sh 'ssh administrator@172.16.20.93 "nohup java -jar petclinic-1.0.0.jar &"'
   }
    
   stage('Functional tests'){
      //build job: 'UFT Test'
   }
    
   stage('Performance tests')
   {
       //build job: 'LR Test Job'
     
   }
    
   stage('Clean up testenvironment'){
       hook2 = registerWebhook()
       

             
       build job: 'oo_remove_runner_vm', parameters: [[$class: 'StringParameterValue', name: 'hook_url', value: hook2.getURL()],[$class: 'StringParameterValue', name: 'vmName', value: vmName]]
       
       echo "Waiting for POST to ${hook2.getURL()}"
       
       data = waitForWebhook hook2
       
       echo "Webhook called with data: ${data}, VM was removed succesfully"
   }
}
