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
        sh 'terraform init -no-color'
      }
    }
    stage('Plan') {
      steps {
        sh 'terraform plan -no-color'
      }
    }
    stage('Validate Apply') {
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
       sh 'terraform apply -auto-approve -no-color'
      }
    }
    stage('EC2 Wait') {
      steps {
        sh 'aws ec2 wait instance-status-ok --region us-east-2'
      }
    }
    stage('Validate EC2 instance'){
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
        sh 'terraform destroy -auto-approve -no-color'
      }
    }
  }
}