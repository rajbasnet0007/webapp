

pipeline {
    environment {
    credentialsId = 'bit1-cred'
    accessKeyVariable = 'AWS_ACCESS_KEY_ID'
    secretKeyVariable = 'AWS_SECRET_ACCESS_KEY'
    registry = "463854429606.dkr.ecr.us-west-2.amazonaws.com/testing"
    registryCredential = 'aws-cred'
    docker_Image="463854429606.dkr.ecr.us-west-2.amazonaws.com/testing:${env.BUILD_NUMBER}"
    TASK_FAMILY="lla-ma-district-services"
    STAGE_TASK_FAMILY="seagull-ut-district-services"
    }
    agent any
    
    stages{
    stage('Deployment Email'){
                    steps{
                      emailext body: 'Build Status: Started to Build Job: ${JOB_NAME}  Build Number: ${BUILD_NUMBER} More info at:  ${BUILD_URL}', subject: 'AudioEnhancement Development', to: 'raj.kumar@appinventiv.com'
                
                    }
            }
            

   stage('Build image') {
    steps{
    script{
    sh '${WORKSPACE}'
    sh 'cd ${WORKSPACE}/webapp/'    
    dockerImage=docker.build(registry + ":$BUILD_NUMBER","-f Dockerfile .")
    sh "echo $docker_Image "
        }
     }
  }
stage('Push Image to Registry') {
    steps{
    script {
    docker.withRegistry("https://" + registry, "ecr:us-west-2:" + registryCredential)
    {
     dockerImage.push()
    }
    }
    }
    }
    stage('Deployment Started On Test Env ') {
            steps {
                sh ''' #!/bin/bash
                echo "New task created for test env"
            TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition "$TASK_FAMILY")
            NEW_TASK_DEFINTIION=$(echo "$TASK_DEFINITION" | jq --arg IMAGE "$docker_Image" '.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.registeredAt) | del(.registeredBy) | del(.requiresAttributes) | del(.compatibilities)')
            NEW_TASK_INFO=$(aws ecs register-task-definition --region "us-west-2" --cli-input-json "$NEW_TASK_DEFINTIION")
            NEW_REVISION=$(echo $NEW_TASK_INFO | jq '.taskDefinition.revision')
            echo $NEW_REVISION
            aws s3 cp s3://ecs-deployment-bucket-1/testing_env/appspec.yaml .
            aws s3 cp s3://ecs-deployment-bucket-1/testing_env/deployment/create-deployment.json .
            sed -i "s/UPC/$NEW_REVISION/g" appspec.yaml
            aws s3 cp appspec.yaml s3://ecs-deployment-bucket-1/testing_env/updated/
            aws deploy create-deployment --cli-input-json file://create-deployment.json --region us-west-2
                '''
            }
    
       }
    stage('Approval Step Deployment For Staging'){
            steps {

	      		// Create an Approval Button with a timeout of 15minutes.
	                timeout(time: 15, unit: "MINUTES") {
	                    input message: 'Do you want to approve the deployment?', ok: 'Yes'
	                }
            }
        }
           stage('Deployment Started On Staging Env ') {
            steps {
                sh ''' #!/bin/bash
                echo "New task created"
            STAGE_TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition "$STAGE_TASK_FAMILY")
            STAGE_NEW_TASK_DEFINTIION=$(echo "$STAGE_TASK_DEFINITION" | jq --arg IMAGE "$docker_Image" '.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.registeredAt) | del(.registeredBy) | del(.requiresAttributes) | del(.compatibilities)')
            STAGE_NEW_TASK_INFO=$(aws ecs register-task-definition --region "us-west-2" --cli-input-json "$STAGE_NEW_TASK_DEFINTIION")
            STAGE_NEW_REVISION=$(echo $STAGE_NEW_TASK_INFO | jq '.taskDefinition.revision')
            echo $STAGE_NEW_REVISION
            aws s3 cp s3://ecs-deployment-bucket-1/staging_env/appspec.yaml .
            aws s3 cp s3://ecs-deployment-bucket-1/staging_env/deployment/create-deployment.json .
            sed -i "s/UPC/$STAGE_NEW_REVISION/g" appspec.yaml
            aws s3 cp appspec.yaml s3://ecs-deployment-bucket-1/staging_env/updated/
            aws deploy create-deployment --cli-input-json file://create-deployment.json --region us-west-2
                '''
            }
    
       }
   }
   post {
        // Clean after build
        always {
                cleanWs deleteDirs: true, patterns: [[pattern: 'node_modules', type: 'EXCLUDE']]
                             
        }
    }
}
