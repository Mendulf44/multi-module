pipeline {
   agent any 

    tools {
    maven 'maven3'
    jdk 'JDK21'
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
                    junit '**/target/surefire-reports/TEST-*.xml'
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
                        echo 'Tests de Vulnérabilités OWASP'
                    }
                    
                }
                 stage('Analyse Sonar') {
                     steps {
                        echo 'Analyse sonar'
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

