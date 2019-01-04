pipeline {
  agent any
  stages {
    stage("Creating stack") {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']],
                               doGenerateSubmoduleConfigurations: false,
                               extensions: [],
                               submoduleCfg: [],
                               userRemoteConfigs: [[credentialsId: 'git-ssh',
                               url: 'https://github.com/lilapetryshyn/Cloudformation.git']]])
        script {
          String stacks = sh(returnStdout: true, script: 'aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE --query "StackSummaries[].StackName"')
          def chooseStack = (stacks.contains('blue')) ? 'green' : 'blue'
          env.stackName = chooseStack
          sh "aws cloudformation create-stack --stack-name ${env.stackName} --template-body file://Instance.yaml ; \
          aws cloudformation wait stack-create-complete --stack-name ${env.stackName}"
          env.INSTANCE_IP = sh(returnStdout: true, script: """aws cloudformation describe-stacks --stack-name ${env.stackName} --query "Stacks[].Outputs[].OutputValue" --output text""").toString().replaceAll('\n', '')
        }
      }
    }
    stage("Check app is available") {
      steps {
        script {
          println env.INSTANCE_IP
          def appUrl = sh(returnStdout: true, script: """curl 'http://${env.INSTANCE_IP}:8080/helloworld/'""")
          if (appUrl.contains('Hello World!')) {
            echo "TEST is SUCCESS"
            if (env.stackName == 'blue') {
              sh "aws cloudformation delete-stack --stack-name green; \
                  aws cloudformation wait stack-delete-complete --stack-name green"
            }
            else {
              sh "aws cloudformation delete-stack --stack-name blue; \
                  aws cloudformation wait stack-delete-complete --stack-name blue"
            }
          }
          else {
            echo "TEST FAILED"
            sh "aws cloudformation delete-stack --stack-name ${env.stackName}; \
                aws cloudformation wait stack-delete-complete --stack-name ${stackName}"
            currentBuild.result = 'FAILED'
          }
        }
      }
    }
  }
  post {
    success {
      slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
    failure {
      slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
  }
}
