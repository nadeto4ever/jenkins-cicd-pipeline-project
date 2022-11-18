def COLOR_MAP = [
        'SUCCESS': 'good', 
        'FAILURE': 'danger',
    ]

pipeline {
    
    environment {
        MESSAGE = "Hello World"

    }
    
    
    agent any
    
    tools {
        maven 'localMaven'
        jdk 'localJdk'
    }

  stages {
        stage('Git checkout') {
            steps {
                echo 'Cloning the application code ....'
                git branch: 'main', url: 'https://github.com/nadeto4ever/jenkins-cicd-pipeline-project.git'
                 sh 'mvn --version'
                 
            }
        }
        
        stage('Build') {
            steps {
            sh 'java -version'
            sh 'mvn clean package' 
            echo "{$MESSAGE}"
            }
            
            post {
            success {
                echo 'archiving....'
                archiveArtifacts artifacts: '**/*.war', followSymlinks: false
            }
        }
        }
        
        stage('Unit Test'){
        steps {
            sh 'mvn test'
        }
    }
        stage('Integration Test'){
            steps {
              sh 'mvn verify -DskipUnitTests'
            }
        }
        stage ('Checkstyle Code Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        stage('SonarQube Scanning') {
          steps {
              
              withSonarQubeEnv('SonarQube') {
              
              sh """mvn sonar:sonar \
              -Dsonar.projectKey=JavaWebApp \
              -Dsonar.host.url=http://172.31.86.44:9000 \
              -Dsonar.login=f3810c95a3f04cdd1510a60b34a1079cade8c86f
                """
              }
          }
        }
        
        stage("Quality Gate"){
          steps{
           waitForQualityGate abortPipeline: true
             }
        }
        
         stage("Upload Artifact to Nexus"){
          steps{
           sh 'mvn clean deploy -DskipTests'
             }
        }
        
         stage('Deploy to DEV') {
          environment {
            HOSTS = "dev"
          }
          steps {
            sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
          }
         }
         
         stage('Deploy to STAGING') {
          environment {
            HOSTS = "stage"
          }
          steps {
            sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
          }
         }
         
    stage('Approval') {
              steps {
                input('Do you want to proceed?')
              }
            }
    stage('Deploy to PROD') {
          environment {
            HOSTS = "prod"
          }
          steps {
            sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
          }
         }
    }
      
    post { 
        always { 
            echo 'I will always say Hello again!'
            slackSend channel: '#glorious-w-f-devops-alerts', color: COLOR_MAP[currentBuild.currentResult], message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
    }

 