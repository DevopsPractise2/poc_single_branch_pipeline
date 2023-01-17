pipeline {
  //triggers{ cron('H/15 * * * *') }
  //properties([pipelineTriggers([cron('* * * * *')])])
  options {
    buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
  }
    agent any 
    stages {
        stage('poll scm') {
            steps {
              script {
                 properties([pipelineTriggers([pollSCM('* * * * *')])])
            }
          }
        }
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/dev']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/sreevasdev/poc_single_branch_pipeline.git']]])
                sh "ls -lart"
            }
        } 
        stage('Build') { 
            steps {
              script {
               try {
                 sh '''/opt/apache-maven-3.6.3/bin/mvn clean install dockerfile:build'''
               } catch(Exception e) {
                 //error "Program failed, please read logs..."
                 echo 'Exception occurred: ' + e.toString()
                 sh 'Handle the exception!'
               }
               //sh '''/opt/apache-maven-3.6.3/bin/mvn clean install dockerfile:build'''
               }
            }
        }
        stage('sonar scan') { 
            steps {
                echo "deploy"
                //sh "/opt/apache-maven-3.6.3/bin/mvn -X sonar:sonar"
                sh "/opt/apache-maven-3.6.3/bin/mvn sonar:sonar \
                                             -Dsonar.projectKey=sonar-project \
                                             -Dsonar.host.url=http://3.17.75.1:9000 \
                                             -Dsonar.login=c1c37715c289c6a3b4aa550b702652dc281ec2d2"
                script {
                   //emailext mimetype: 'text/html', subject: "last success", to: "sreevasdev78@gmail.com"
                   //emailext body: 'text/html', subject: "last success", to: "sreevasdev78@gmail.com"
                   mail bcc: '', body: 'Scanned the code, Please find the scanning reports', cc: '', from: '', replyTo: '', subject: 'Scanning the code', to: 'sreevasdev78@gmail.com'
                   
                   }
            }
        }
        stage('aws'){
            steps{
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-jenkins',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        sh "aws ec2 describe-instances --region=us-east-1"
                    }
            }
        }
        stage('push Image to ECR'){ 
            steps {
               // withEnv (["AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}", "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}", "AWS_DEFAULT_REGION=${env.AWS_DEFAULT_REGION}"]) {
                    //sh "docker login -u AWS -p ${aws ecr-public get-login-password --region us-east-1} public.ecr.aws/a7j1t7p3"
                    //sh '''docker build -t newecr .'''
                    //sh '''docker tag newecr:""$BUILD_ID"" newecr:""$BUILD_ID"" '''
                    //sh '''docker push public.ecr.aws/a7j1t7p3/newecr:latest'''
                    //sh ''' aws ecr get-login-password --region region | docker login --username AWS --password-stdin 701796411100.dkr.ecr.us-east-1.amazonaws.com/sample_poc '''
                    sh ''' docker login -u AWS -p $(aws ecr get-login-password --region us-east-1) 701796411100.dkr.ecr.us-east-1.amazonaws.com/sample_poc '''
                    sh '''/opt/apache-maven-3.6.3/bin/mvn clean install dockerfile:build'''
                    sh '''docker tag hello-howtodoinjava/hello-docker:latest  701796411100.dkr.ecr.us-east-1.amazonaws.com/sample_poc:latest'''
                    sh '''docker push  701796411100.dkr.ecr.us-east-1.amazonaws.com/sample_poc:latest'''
              /*script {
                  
                docker.withRegistry("public.ecr.aws/a7j1t7p3/newecr", "ecr:us-east-1:credential-id") {
                    docker.image("your-image-name").push()
                  }
                  
                  docker.withRegistry('public.ecr.aws/a7j1t7p3/newecr', 'ecr:us-east-1:aws-credentials') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
                  }
              }*/
            //}
            }
        }
        stage('Deploy to S3') { 
            steps {
                echo "deploy"
                sh ''' touch sample.txt'''
                sh '''ls -lrt'''
                sh ''' pwd '''
                sh '''aws s3 cp . s3://sreevassample/ --recursive'''
                script {
                   //emailext mimetype: 'text/html', subject: "last success", to: "sreevasdev78@gmail.com"
                   emailext body: 'text/html', subject: "last success", to: "sreevasdev78@gmail.com"
                   
                   }
            }
        }
    }
}
