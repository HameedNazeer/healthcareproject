pipeline {
agent any 
tools {
maven 'maven'
}

stages {
stage("Git Checkout"){
steps{
git 'https://github.com/HameedNazeer/healthcareproject.git'
 }
 }
stage('Build the application'){
steps{
echo 'Building the package using mvn'
sh 'mvn clean install package'
 }
 }
 
stage('generate report'){
steps{
    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/healthcare/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
 }
}
stage('Docker build image') {
              steps {
                  
                  sh'sudo docker system prune -af '
                  sh 'sudo docker build -t hameedn/health_care:3.0 . '
              
                }
            }
stage('Docker login and push') {
              steps {
                   withCredentials([string(credentialsId: 'docker', variable: 'dockerpassword')]) {
                  sh 'sudo docker login -u hameedn -p ${dockerpassword} '
                  sh 'sudo docker push hameedn/health_care:3.0'
                  }
                }
        }    
 stage (' setting up Kubernetes with terraform '){
            steps{

                dir('terraform_files'){
                sh 'terraform init'
                sh 'terraform validate'
                sh 'terraform apply --auto-approve'
                sh 'sleep 20'
                }
               
            }
        }
stage('deploy application to kubernetes'){
  steps{
    sh 'sudo chmod 400 ./terraform_files/mumbaikey.pem'
    sh 'sudo scp -o StrictHostKeyChecking=no -i ./terraform_files/mumbaikey.pem deployment.yml ubuntu@3.109.14.87:/home/ubuntu/'
    sh 'sudo scp -o StrictHostKeyChecking=no -i ./terraform_files/mumbaikey.pem service.yml ubuntu@3.109.14.87:/home/ubuntu/'
    sh 'ssh -i ./terraform_files/mumbaikey.pem  ubuntu@3.109.14.87 minikube start'
    script{
    try{
    sh 'ssh -i ./terraform_files/mumbaikey.pem  ubuntu@3.109.14.87 kubectl apply -f .'
    }catch(error)
    {
    sh 'ssh -i ./terraform_files/mumbaikey.pem  ubuntu@3.109.14.87 kubectl apply -f .'
}
}
}
}
}
}
