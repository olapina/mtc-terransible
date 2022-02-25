pipeline {
  agent any
  environment {
    TF_IN_AUTOMATION = 'true'
    TF_CLI_CONFIG_FILE = credentials('tf-creds')
    AWS_SHARED_CREDENTIALS_FILE='/home/ubuntu/.aws/credentials'
  }
  stages {
    stage('Init') {
      steps {
        sh 'ls'
        sh 'cat $BRANCH_NAME.tfvars'
        sh 'terraform init -no-color'
      }
    }
    stage('Plan') {
      steps {
        sh 'terraform plan -var-file="$BRANCH_NAME.tfvars" -no-color'
      }
    }
    stage('Validate Apply') {
      when {
        beforeInput true
        branch "dev"
      }
      input {
        message "Do you want to apply this plan?"
        ok "Apply this plan."
      }
      steps {
        echo "Apply accepted"
      }
    }
    stage('Apply') {
      steps {
       sh 'terraform apply -auto-approve -var-file="$BRANCH_NAME.tfvars" -no-color'
      }
    }
    stage('Inventory') {
      steps {
        sh '''printf \\ 
          "\\n$(terraform output -json instance_ips | jq -r \'.[]\')" \\
          >> aws_hosts'''
      }
    }
    stage('EC2 Wait') {
      steps {
        sh '''aws ec2 wait instance-status-ok \\
          --instance-ids $(terraform show -json | jq -r \'.values\'.\'root_module\'.\'resources[] | select (.type == "aws_instance").values.id\') \\
          --region us-east-2'''
      }
    }
    stage('Validate EC2 instance'){
      when {
        beforeInput true
        branch "dev"
      }
      input {
        message "Do you want to run Ansible?"
        ok "Running Ansible on EC2 instance"
      }
      steps {
        echo "Response accepted"
      }
    }
    stage('Ansible') {
      steps {
        ansiblePlaybook(credentialsId: 'ec2-ssh-key', inventory: 'aws_hosts', playbook: 'playbooks/main_playbook.yml')
      }
    }
    stage('Validate Ansible run') {
      input {
        message "Proceed to Destroy?"
        ok "Proceeding to Destroy"
      }
      steps {
        echo "Response accepted"
      }
    }
    stage('Destroy') {
      steps {
        sh 'terraform destroy -auto-approve -var-file="$BRANCH_NAME.tfvars" -no-color'
      }
    }
  }
  post {
    success {
    echo "Success!"
    }
    failure {
      sh 'terraform destroy -auto-approve -var-file="$BRANCH_NAME.tfvars" -no-color'
    }
    aborted {
      sh 'terraform destroy -auto-approve -var-file="$BRANCH_NAME.tfvars" -no-color'
    }
  }
}