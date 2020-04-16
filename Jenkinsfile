pipeline {
  agent any 
  tools {
    maven 'Maven'
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
 /*   
    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'ls /usr/local/bin'
        sh 'sudo /usr/local/bin/trufflehog --json https://github.com/haythemarfaoui/webapp.git | tee trufflehog'
        sh 'cat trufflehog'
      }
    } */
   /* 
    stage ('Source Composition Analysis') {
      steps {
         dependencyCheck additionalArguments: '', odcInstallation: 'Dependency', skipOnScmChange: true
         dependencyCheckPublisher pattern: 'dependency-check-report.xml'
      }
    } */
  /*  
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    } */
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
    
    stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'cp target/*.war /opt/tomcat/latest/webapps/webapp.war'
              }      
           }       
    }
    
/*    
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@13.232.158.44 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://13.232.202.25:8080/webapp/" || true'
        }
      }
    }
    */
  }
}
