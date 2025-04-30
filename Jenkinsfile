def integrationURL = ''
def datacenters = []

def checkSonarQualityGate(){
    // Get properties from report file to call SonarQube 
    def sonarReportProps = readProperties  file: 'target/sonar/report-task.txt'
    def sonarServerUrl = sonarReportProps['serverUrl']
    def ceTaskUrl = sonarReportProps['ceTaskUrl']
    def ceTask

    // Get task informations to get the status
    timeout(time: 4, unit: 'MINUTES') {
        waitUntil(initialRecurrencePeriod: 1000)  {
            withCredentials ([string(credentialsId: 'SONAR_TOKEN', variable : 'token')]) {
                def response = sh(script: "curl -u ${token}: ${ceTaskUrl}", returnStdout: true).trim()
                ceTask = readJSON text: response
            }

            echo ceTask.toString()
              return "SUCCESS".equals(ceTask['task']['status'])
        }
    }

    // Get project analysis informations to check the status
    def ceTaskAnalysisId = ceTask['task']['analysisId']
    def qualitygate

    withCredentials ([string(credentialsId: 'SONAR_TOKEN', variable : 'token')]) {
        def response = sh(script: "curl -u ${token}: ${sonarServerUrl}/api/qualitygates/project_status?analysisId=${ceTaskAnalysisId}", returnStdout: true).trim()
        qualitygate =  readJSON text: response
    }

    echo qualitygate.toString()
    if ("ERROR".equals(qualitygate['projectStatus']['status'])) {
        error "Quality Gate failure"
    }
}

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
                    stash includes: 'application/target/*.jar', name: 'application_main'
                }
                failure{
                    mail bcc: '', body: 'Test 1 - 2.2.2', cc: 'mael.marchand@protonmail.com', from: '', replyTo: '', subject: 'Test 1 - 2.2.2', to: 'mael.marchand@bnpparibas.com'
                } 
            }
             
        }
        
        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    agent any    
                    steps {

                    // Run Maven on a Unix agent.
                    sh 'mvn -DskipTests verify'
                    }
                    
                }
                 stage('Analyse Sonar') {
                    agent any
                    steps {
                        // Run Maven on a Unix agent.
                        sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                    }
                    
                }
            }
            
        }
   
        stage('Reading Configuration') {
            agent any

            steps {
                script {
                    def props = readJSON file: 'deployment.json'
                    integrationURL = props['integrationURL']
                    datacenters = props['dataCenters']
                    for (datacenter in datacenters) {  
                    echo "${datacenter}"
                    }
                }    
            }
        }
 
        stage('Deploiement integration') {
            agent any
            
            when {
                 checkSonarQualityGate=true
                                
            }

            input { message "Voulez-vous deployer"
                ok "Yes"
                }

            steps {
                echo "Continue"  
               }   
                }    
        }

        stage('Deploiement Sur Dcs') {
            agent any

            steps {
                unstash 'application_main'
                script {
                     
                    for (datacenter in datacenters) {  
                    sh "cp application/target/*.jar ${integrationURL}/${datacenter}.jar"
                    }
                }   
                echo "Déploiement dans tous les DCs"


                }
        }

     }
    
}

