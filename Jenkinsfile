pipeline {
  agent {
    kubernetes {
      label 'jenkins-docker-job'
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
    stage('Docker version'){
      steps {
        container('jenkins-docker-agent'){
            withCredentials([[
              $class: 'AmazonWebServicesCredentialsBinding',
              credentialsId: "${CFN_CREDENTIALS_ID}",
              accessKeyVariable: 'AWS_ACCESS_KEY_ID',
              secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]){

                sh 'docker version'
               
              }
        }
      }
    }
    stage('Docker pull'){
      steps {
        container('jenkins-docker-agent'){
            withCredentials([[
              $class: 'AmazonWebServicesCredentialsBinding',
              credentialsId: "${CFN_CREDENTIALS_ID}",
              accessKeyVariable: 'AWS_ACCESS_KEY_ID',
              secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]){

                sh 'sudo docker pull dwolla/jenkins-agent-awscli'
                sh 'sudo aws ecr get-login-password --region us-east-1 | sudo docker login --username AWS --password-stdin 825802405308.dkr.ecr.us-east-1.amazonaws.com'
                sh 'sudo docker tag dwolla/jenkins-agent-awscli:latest 825802405308.dkr.ecr.us-east-1.amazonaws.com/jenkins-agent-awscli:latest'
                sh 'sudo docker push 825802405308.dkr.ecr.us-east-1.amazonaws.com/jenkins-agent-awscli:latest'
               
              }
        }
      }
    }
  }
}
