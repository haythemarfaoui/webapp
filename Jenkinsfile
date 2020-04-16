pipeline {
  agent any 
  tools {
    maven 'maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
   
    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run --rm -v $PWD:/target yellowmegaman/container-trufflehog --regex --entropy=False https://github.com/haythemarfaoui/webapp.git | tee trufflehog'
        sh 'cat trufflehog'
      }
    } 
    /*
    stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/haythemarfaoui/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
        
      }
    } */
   
    stage ('Source Composition Analysis') {
      steps {
          dependencyCheck additionalArguments: '', odcInstallation: 'dependency'
          dependencyCheckPublisher pattern: 'dependency-check-report.xml'
       }
    }
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    } 
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
    
    stage ('Deploy-To-Tomcat') {
           steps {
                sh 'cp target/*.war /opt/tomcat/latest/webapps/webapp.war'      
           }       
    }
    
   
    stage ('DAST') {
      steps {
        sh 'docker run --name zap -u zap -p 2375:2375 -d owasp/zap2docker-stable zap.sh -daemon -port 2375 -host 127.0.0.1 -config api.disablekey=true -config scanner.attackOnStart=true -config view.mode=attack -config connection.dnsTtlSuccessfulQueries=-1 -config api.addrs.addr.name=.* -config api.addrs.addr.regex=true'
        sh 'docker exec zap zap-cli -p 2375 status -t 120'
        sh 'docker exec zap zap-cli -p 2375 open-url http://localhost:9090/webapp/'
        sh 'docker exec zap zap-cli -p 2375 spider http://localhost:9090/webapp/'
        sh 'docker exec zap zap-cli -p 2375 active-scan -r http://localhost:9090/webapp/'
        sh 'docker exec zap zap-cli -p 2375 alerts'
        sh '''
        divider==================================================================
        printf "\n"
        printf "$divider"
        printf "ZAP-daemon log output follows"
        printf "$divider"
        printf "\n"
        '''
        sh 'docker logs zap'
        sh 'docker stop zap'
        sh 'docker rm zap'
      }
    }
    
  }
}
