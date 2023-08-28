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

                sh 'echo "version"'
               
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
                sh 'uname -a'
                sh 'sudo yum update -y'
                sh 'sudo amazon-linux-extras install docker'
                sh 'sudo yum install -y docker'
                sh 'sudo service docker start"' 
                sh 'sudo docker version'


                sh 'docker pull dwolla/jenkins-agent-awscli'
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 825802405308.dkr.ecr.us-east-1.amazonaws.com'
                sh 'docker tag dwolla/jenkins-agent-awscli:latest 825802405308.dkr.ecr.us-east-1.amazonaws.com/jenkins-agent-awscli:latest'
                sh 'docker push 825802405308.dkr.ecr.us-east-1.amazonaws.com/jenkins-agent-awscli:latest'
               
              }
        }
      }
    }
  }
}
