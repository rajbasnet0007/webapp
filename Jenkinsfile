// pipeline {
//     agent any

//     stages {
//         stage('Build') {
//             steps {
//                 echo 'Building..'
//             }
//         }
//         stage('Test') {
//             steps {
//                 echo 'Testing..'
//             }
//         }
//         stage('Deploy') {
//             steps {
//                 echo 'Deploying....'
//             }
//         }
//     }
// }
pipeline {
    environment {
    credentialsId = 'jenkins-demo'
    accessKeyVariable = 'AWS_ACCESS_KEY_ID'
    secretKeyVariable = 'AWS_SECRET_ACCESS_KEY'
    registry = "463854429606.dkr.ecr.us-west-2.amazonaws.com/testing"
    registryCredential = 'jenkins-demo'
    docker_Image="463854429606.dkr.ecr.us-west-2.amazonaws.com/testing:${env.BUILD_NUMBER}"
    
    }
    agent any
    
    stages{
    stage('EmailProd'){
                    steps{
                        emailext body: "<body><p><font size='+2'><b>Build Status: </b>Started <br> <b>Build Job: </b> ${env.JOB_NAME} <br><b> Build Number: </b> ${env.BUILD_NUMBER} </font> <br><br> <font size='+1'>More info at:  ${env.BUILD_URL}</font></p></body>", subject: "AE Development : Jenkins Build Job : ${env.JOB_NAME}", to: 'raj.kumar@appinventiv.com'
                
                    }
            }
    
    stage('Clone repository') {
            steps{
    checkout scm
        }
   }
   }
}
