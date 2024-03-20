pipeline {
agent any
stages {
stage('GIT CHECKOUT') {
steps {
  git branch: 'CI_CD_Github_Actions', credentialsId: 'GIT', url: 'https://github.com/iincube/ServerV2.git'
}
}
stage('Gradle build') {
steps {
sh 'java -version'
sh '/opt/gradle/gradle-4.4/bin/gradle -version'
sh 'cd mongo_rest && /opt/gradle/gradle-4.4/bin/gradle :safetraxrest:distzip'
sh 'ls mongo_rest/safetraxrest/build/distributions'
}
}
stage('STORE ARTIFACTS IN GCP') {
steps {
script {
def projectId = 'academic-elixir-90710'
def bucketName = 'test-live-deploy'
sh 'gsutil cp /var/lib/jenkins/workspace/SERVER-AUTOMATION/mongo_rest/safetraxrest/build/distributions/safetraxrest.zip gs://test-live-deploy/SERVERV2' 
//sh "gcloud compute ssh --zone=asia-south1-c -q --ssh-key-file=/var/lib/jenkins/id_rsa bhavanisankar@cicd-test --command 'gsutil cp gs://test-live-deploy/SERVERV2/safetraxrest.zip .'"
}
}
}
stage('Deploy to Another Server in GCP') {
steps {
script {

//sh "gcloud compute ssh --zone=asia-south1-c -q --ssh-key-file=/var/lib/jenkins/id_rsa @cicd-test --command 'sh /home/bhavanisankar/ETS-mongoser-deployment-updated.sh'"
//sh 'ssh -i  /var/lib/jenkins/id_rsa bhavanisankar@cicd-test -o StrictHostKeyChecking=no "sudo sh /home/bhavanisankar/ETS-mongoser-deployment-updated.sh"'
sh 'ssh -i /var/lib/jenkins/id_rsa venu@cicd-test -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no "sudo /home/venu/ETS-mongoser-deployment-updated.sh"'

             }
                
           }
        }
    }
}
