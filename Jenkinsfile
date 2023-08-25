pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '85e3b934-3460-4b99-9e0f-66fe1a805e16', url: 'https://github.com/akannan1087/myJuly2023Weekday']])
            }
        }
        
        stage ("Build") {
            steps {
                sh "mvn clean install -f MyWebApp/pom.xml"      
            }
        }
        
        stage ("Code scan") {
            steps {
                withSonarQubeEnv("SonarQube") {
                    sh "mvn sonar:sonar -f MyWebApp/pom.xml"      
                }
            }
        }
        
        stage ("coverage") {
            steps {
                jacoco()
            }
        }
        
        stage ("Nexus upload") {
            steps {
                    nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: '34594541-7add-46d5-83d6-8ef8a7e6774f', groupId: 'com.capone.af', nexusUrl: 'ec2-54-157-179-191.compute-1.amazonaws.com:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            }
        }
        
        stage ("Dev Deploy") {
            steps {
                deploy adapters: [tomcat9(credentialsId: '9416e094-3099-4719-988d-5d828d0d5f21', path: '', url: 'http://ec2-54-86-227-12.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage ("Notify") {
            steps {
                slackSend channel: 'july-2023-weekday-batch', message: 'DEV Deploy was done, please start smoke testing in DEV env'
            }
        }
        
        stage ("DEV approve") {
            steps {
                    echo "Taking approval from DEV Manager for QA Deployment"     
                    timeout(time: 8, unit: 'HOURS') {
                    input message: 'Do you approve QA Deployment?', submitter: 'admin, dev-manager@email.com, dev-mgr2@gmail.com'
            }
          }
        }
        
        stage ("QA Deploy") {
            steps {
                deploy adapters: [tomcat9(credentialsId: '9416e094-3099-4719-988d-5d828d0d5f21', path: '', url: 'http://ec2-54-86-227-12.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
            }
         }
         
        stage ("QA notify") {
            steps {
                slackSend channel: 'july-2023-weekday-batch,qa-testing-team', message: 'QA Team - QA deployment was done, please start your functional testing in QA env..'
           }
        }
        
        stage ("QA approve") 
        {
            steps {
                echo "Taking approval from QA Manager for PROD Deployment"     
                timeout(time: 4, unit: 'DAYS') {
                input message: 'Do you approve PROD Deployment?', submitter: 'admin'
                }
            }
        }
        
        stage ("PROD Deploy") {
            steps {
                deploy adapters: [tomcat9(credentialsId: '9416e094-3099-4719-988d-5d828d0d5f21', path: '', url: 'http://ec2-54-86-227-12.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
            }
        }
        stage ("Final notify") {
            steps {
            slackSend channel: 'july-2023-weekday-batch,qa-testing-team,product-owners-teams', message: 'product owner - PROD deployment was done, please inform end customers..'
         }
       }
    }
}
