pipeline {
  agent {
    node {
      label 'terraform'
    }
  }
  environment {
    AWS_ACCESS_KEY     = credentials("main_aws_access_key")
    AWS_SECRET_KEY     = credentials("main_aws_secret_key")
    JFROG_ACCESS_TOKEN = credentials("prod/jfrog/jfrog_jenkins/access_token")
  }

  stages {
    // [START tf-init, tf-validate]
    stage('TF Init & Validate') {
      when { anyOf {branch "main";changeRequest() } }
      steps {
        ansiColor('xterm') {
        sh '''
        if [[ $CHANGE_TARGET ]]; then
            TARGET=$CHANGE_TARGET
        else
            TARGET=$BRANCH_NAME
        fi
        if [ -d "tf/${TARGET}/" ]; then
            cd tf/${TARGET}
            terraform init \
              -var "aws_access_key=${AWS_ACCESS_KEY}" \
              -var "aws_secret_key=${AWS_SECRET_KEY}"
            terraform validate
        else
          echo "Target branch does not exist"
          exit 0
        fi'''
        }
      }
    }
    // [END tf-init, tf-validate]

    // [START tf-plan]
    stage('TF Plan') {
      when { anyOf {branch "main";changeRequest() } }
      steps {
        ansiColor('xterm') {
        sh '''
        if [[ $CHANGE_TARGET ]]; then
            TARGET=$CHANGE_TARGET
        else
            TARGET=$BRANCH_NAME
        fi
        if [ -d "tf/${TARGET}/" ]; then
            cd tf/${TARGET}
            terraform plan \
              -var "aws_access_key=${AWS_ACCESS_KEY}" \
              -var "aws_secret_key=${AWS_SECRET_KEY}"
        else
          echo "Target branch does not exist"
          exit 0
        fi'''
        }
      }
    }
    // [END tf-plan]

    stage('Approval') {
        when { anyOf {branch "main";} }
        steps {
          script {
            def userInput = input(id: 'confirm', message: 'Apply Terraform?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Apply terraform', name: 'confirm'] ])
          }
        }
      }

    // [START tf-apply]
    stage('TF Apply') {
      when { anyOf {branch "main";} }
      steps {
        ansiColor('xterm') {
        sh '''
        TARGET=$BRANCH_NAME
        if [ -d "tf/${TARGET}/" ]; then
            cd tf/${TARGET}
            terraform apply -input=false -auto-approve \
              -var "aws_access_key=${AWS_ACCESS_KEY}" \
              -var "aws_secret_key=${AWS_SECRET_KEY}"
        else
            echo "****** SKIPPING APPLY ******"
            echo "Branch '$TARGET' does not represent an official directory."
            echo "****** APPLY SKIPPED *******"
        fi'''
        }
      }
    }
    // [END tf-apply]

    // [START tf-show]
    stage('TF Show') {
      when { anyOf {branch "main";} }
      steps {
        ansiColor('xterm') {
        sh '''
        TARGET=$BRANCH_NAME
        if [ -d "tf/${TARGET}/" ]; then
            cd tf/${TARGET}
            terraform show
        else
            echo "***** SKIPPING TFSHOW ******"
        fi'''
        }
      }
    }
    // [END tf-show]
  }
}
