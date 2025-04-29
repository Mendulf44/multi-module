pipeline {
   agent any 

    tools {
    maven 'maven3'
    jdk 'JDK21'
    }

    environment {
        SONAR_TOKEN=credentials('SONAR_TOKEN_ID')

    }

    stages {
        stage('Build and Test') {
            steps {

                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }


            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                always{
                    junit '**/target/surefire-reports/*.xml'
                } 
                success {
                    archiveArtifacts 'application/**/*.jar'
                }
                failure{
                    mail bcc: '', body: 'Test 1 - 2.2.2', cc: 'mael.marchand@protonmail.com', from: '', replyTo: '', subject: 'Test 1 - 2.2.2', to: 'mael.marchand@bnpparibas.com'
                } 
            }
             
        }
        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
            steps {

                // Run Maven on a Unix agent.
                sh 'mvn -DskipTests verify'
                }
                    
                }
                 stage('Analyse Sonar') {
            steps {
                // Run Maven on a Unix agent.
                sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                }
                    
                }
            }
            
        }
            
        stage('Déploiement intégration') {

            steps {
                echo "Déploiement intégration"
                
            }
        }

     }
    
}

