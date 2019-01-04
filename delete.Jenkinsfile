import groovy.util.Eval
pipeline {
  agent any
  stages {
    stage("Delete all stacks") {
      steps {
        script {
          def stacks = sh(returnStdout: true, script: 'aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE --query "StackSummaries[].StackName"').replaceAll('\n', '')
          def stackList = Eval.me(stacks)
          println stackList
          for (stack in stackList) {
            sh "aws cloudformation delete-stack --stack-name ${stack}; \
                aws cloudformation wait stack-delete-complete --stack-name ${stack}"
          }
        }
      }
    }
  }
}
