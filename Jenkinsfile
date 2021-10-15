pipeline{
    agent any
    stages {
        stage('Build Backend'){
            steps{
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit tests'){
            steps{
                bat 'mvn test'
            }
        }
         stage('Sonar Analysis'){
             environment{
                 scannerHome = tool 'SONAR_SCANNER'
             }
            steps{
                withSonarQubeEnv('SONAR_LOCAL'){
              bat "\"${scannerHome}\\bin\\sonar-scanner.bat\" -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=d6fe6f2dfa5c59b9d597758cde721ca17faf1b1a -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
         }
            stage('Quality Gate'){
            steps{
               sleep(5)
               timeout(time: 1, unit: 'MINUTES'){
                waitForQualityGate abortPipeline: true
               }
            }
            }
             stage('Deploy Backend'){
            steps{
         deploy adapters: [tomcat8(credentialsId: 'TomcatLogn', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'            }
             }
            stage('API Test'){
            steps{
                   dir('api-test') {
              git 'https://github.com/marcospaulo7/tasks-api-test'
              bat 'mvn test'
                   }
            }
            }

        stage('Deploy Frontend'){
            steps{
              dir('frontend') {
              git 'https://github.com/marcospaulo7/tasks-frontend'
              bat 'mvn clean package'
         deploy adapters: [tomcat8(credentialsId: 'TomcatLogn', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
              }
            }
	}
	         stage ('Functional Test') {
            steps {
                dir('funcional-test') {
                    git credentialsId: '82a16b13-f792-4078-b8bd-7df90498c75c', url: 'https://github.com/marcospaulo7/tasks-funcional-tests'
                    bat 'mvn test'
                }
            }
        }
	    
	         stage ('Deploy Prod') {
            steps {
                echo 'docker-compose build'
                echo 'docker-compose up -d'
            }
        } 
	    
	    post {
		    always{
		    junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml'
		    }
		    unsuccessful{
		    emailext attachLog: true, body: 'see the attached log below', subject: 'Build $BUILD_NUMBER has failed', to: 'marcos.sbfj01d2+jenkins@gmail.com'
		    }
		      fixed{
		    emailext attachLog: true, body: 'see the attached log below', subject: 'Build is fine', to: 'marcos.sbfj01d2+jenkins@gmail.com'
		    }
	    }
    }
}
