pipeline {
  environment {
    imagename = "norbin/train"
    registryCredential = 'dockerhub'
    dockerImage = ''
  }
  agent any
  stages {
    stage('Build') {
      steps {
        echo 'Running build automation'
        sh './gradlew build --no-daemon'
        archiveArtifacts artifacts: 'dist/trainSchedule.zip'
     }
    }
    stage('Cloning Git') {
      steps {
        git([url: 'https://github.com/Norbinsh/cicd-pipeline-train-schedule-dockerdeploy.git', branch: 'master'])

      }
    }
    stage('Building Docker image') {
      steps{
        script {
          dockerImage = docker.build imagename
        }
      }
    }
    stage('Deploy Docker Image To Dockerhub') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push("$BUILD_NUMBER")
             dockerImage.push('latest')

          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $imagename:$BUILD_NUMBER"
         sh "docker rmi $imagename:latest"

      }
    }
    stage('Deploy To Production') {
      steps {
        input 'are you sure?'
        milestone(1)
         kubernetesDeploy(
           kubeconfigId: 'kubeconfig',
           configs: 'train-manifest.yml',
           enableConfigSubstitution: true
        )
      }
    }
  }
}
