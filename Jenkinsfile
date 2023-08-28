pipeline {
  agent {
    kubernetes {
      label 'jenkins-backup-job'
      defaultContainer 'jnlp'
      yamlFile 'k8s/k8sPodTemplate.yaml'
    }
  }
parameters {
    credentials(name: 'CFN_CREDENTIALS_ID', defaultValue: '', description: 'AWS Account Role.', required: true)
    choice(
      name: 'REGION',
      choices: [
          'us-east-1',
          'us-east-1',
          'us-east-2'
          ],
      description: 'AWS Account Region'
    )
    booleanParam(name: 'TOGGLE', defaultValue: false, description: 'Are you sure you want to perform this action?')
  }
  options {
    buildDiscarder(logRotator(numToKeepStr:'30'))
    timeout(time: 60, unit: 'MINUTES')
  }
stages {
    stage('Backup Jenkins'){
      steps {
        container('jenkins-awscli-agent'){
            withCredentials([[
              $class: 'AmazonWebServicesCredentialsBinding',
              credentialsId: "${CFN_CREDENTIALS_ID}",
              accessKeyVariable: 'AWS_ACCESS_KEY_ID',
              secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]){
                sh 'aws --version'
                sh 'aws s3 ls'
                sh 'ls -lts /var'
              sh '''
              echo 'Install kubectl'
              curl -LO "https://storage.googleapis.com/kubernetes-release/release/\$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
              chmod +x ./kubectl
              mv ./kubectl /usr/local/bin/kubectl

              echo 'Create jenkins backup'

              function get_jenkins_pod_id {
                kubectl get pods -n default -l app.kubernetes.io/component=jenkins-master -o custom-columns=PodName:.metadata.name | grep jenkins-
              }
  
              kubectl exec  $(get_jenkins_pod_id) -- bash -c 'cd /var; ls -ltr ;rm -rf /tmp/jenkins_backup; mkdir -p /tmp/jenkins_backup; cp -r jenkins_home /tmp/jenkins_backup/jenkins_home; tar -zcvf /tmp/jenkins_backup/jenkins_backup.tar.gz /tmp/jenkins_backup/jenkins_home'
              
              cd && kubectl cp default/$(get_jenkins_pod_id):/tmp/jenkins_backup/jenkins_backup.tar.gz jenkins_backup.tar.gz
              
              echo 'Upload jenkins_backup.tar to S3 bucket'
              aws s3 cp jenkins_backup.tar.gz s3://dx-devops-backup/$(date +%Y%m%d%H%M)/jenkins_backup.tar.gz
                         
              echo 'Remove files after succesful upload to S3'
              kubectl exec $(get_jenkins_pod_id) -- bash -c 'rm -rf /tmp/jenkins_backup'
              '''
              }
          
        }
      }
    }
  }
}
