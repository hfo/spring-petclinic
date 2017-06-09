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
    
    stage('Cobertura'){
        //sh './mvnw cobertura:check'
    }
   
    stage('Maven-Build') {
        sh './mvnw clean install -Dmaven.test.skip=false'
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
   stage('Funktional Test'){
      build job: 'UFT Test'
   }
   stage('PerformanceTest')
    {
       build job: 'LR Test Job'
     
    }
   
   stage('Nexus Upload Docker Image'){
       sh 'docker tag petclinic_alpine 172.16.20.157:8082/petclinic_alpine'
       sh 'docker push 172.16.20.157:8082/petclinic_alpine'
   }
   
   stage('Deploy'){
       //sh 'docker pull 172.16.20.157:8082/petclinic_alpine' 
       //sh 'docker run -d -p 4100:8080 petclinic_alpine java -jar /usr/src/petclinic/petclinic-1.0.0.jar'
       
       sh 'ssh administrator@172.16.20.159 "docker run --rm -d -p 4000:8080 172.16.20.157:8082/petclinic_alpine java -jar /usr/src/petclinic/petclinic-1.0.0.jar"'
       
       //alternatively deploy via ssh ==> does not close the pipeline atm
       
       //sh 'ssh administrator@172.16.20.93 "rm -f petclinic-1.0.0.jar; wget http://172.16.20.92:8081/repository/Jenkins-Repo/de/proficom/cdp/petclinic/1.0.0/petclinic-1.0.0.jar; ls"'
       //sh 'ssh administrator@172.16.20.93 "nohup java -jar petclinic-1.0.0.jar &"'
    }
}
