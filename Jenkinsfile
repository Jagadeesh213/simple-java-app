#!/bin/bash
pipeline {
  agent any
  tools {
    maven 'M2_HOME'
  }
  stages {
         stage ('Initialize') {
             steps {
                 sh '''
                     M2_HOME=/opt/maven
                     M2=/opt/maven/bin
                     PATH=$PATH:$HOME/bin/:$JAVA_HOME:$M2:$M2_HOME
                     export PATH
                     echo "PATH = ${PATH}"
                     echo "M2_HOME = ${M2_HOME}"
                     whoami
                 '''
             }
         }
    stage('Build app') {
      steps {
        sh 'mvn clean install package'
      }
    stage('SonarQube')
    environment {
        SONAR_CREDENTIALS = credentials('81b8d47a-b775-4c84-bd93-9576bd8da492')
    }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool name: 'SonarQube', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    
                    withCredentials([usernamePassword(credentialsId: '81b8d47a-b775-4c84-bd93-9576bd8da492', usernameVariable: 'admin', passwordVariable: 'sonar@123')]) {
                        def scannerArgs = [
                            "-Dsonar.projectKey=your_project_key",
                            "-Dsonar.sources=src",
                            // Add more parameters as needed
                            "-Dsonar.login=${admin}",
                            "-Dsonar.password=${sonar@123}"
                        ]

                        withSonarQubeEnv('SonarQube') {
                            sh "${scannerHome}/bin/sonar-scanner ${scannerArgs.join(' ')}"
                        }
                    }
                }
            }
        }
    post {
        always {
            // Cleanup or finalization tasks if needed
        }
    } 
//      post {
//        failure {
//            script {
                // Rollback to the previous successful build
//                def previousBuild = currentBuild.getPreviousSuccessfulBuild(205)
//                if (previousBuild) {
//                    echo "Rolling back to the previous successful build: 205,
//                    buildjob: 208, parameters: [[$class: 'IntParameterValue', name: 'DiagnosticsPup', value: 205]]
//                } else {
//                    echo "No previous successful build found. The pipeline cannot be rolled back."
//                }
//            }
//        }
//    }
        post{
        failure{
            emailext to: "jagadeesh.j@apollohl.com",
            subject: "jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}",
            body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}\nMore Info can be found here: ${env.BUILD_URL}"
        }
        changed{
            emailext to: "jagadeesh.j@apollohl.com",
            subject: "jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}",
            body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}\nMore Info can be found here: ${env.BUILD_URL}"
        }    
        success{
            emailext to: "jagadeesh.j@apollohl.com",
            subject: "jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}",
            body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}\nMore Info can be found here: ${env.BUILD_URL}"
        }
      }
    }
    stage('Deploy to Tomcat') {
            steps {
                // SSH into the remote server and deploy the WAR file to Tomcat
                sshagent(['4ec2bb2f-879a-4dab-b5fb-0404a7f7baa4']){
                //sh " scp -v -o StrictHostKeyChecking=no sample/webapp/target/webapp/*.war dev_user@3.109.231.32:/opt/tomcat/qa_webapps/sample"
                sh "scp /var/lib/jenkins/workspace/sample/webapp/target/webapp.war dev_user@3.109.231.32:/opt/tomcat/qa_webapps/sample/"
                }
            }
        }    
//   stage('gitlab') {
//          steps {
//            echo 'Notify GitLab'
//             updateGitlabCommitStatus name: 'build', state: 'pending'
//            updateGitlabCommitStatus name: 'build', state: 'success'
//          }
//        }
    }
}
