pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "gangzdh/train-schedule"       
        BRANCH_NAME = "${GIT_BRANCH.split('/').size() > 1 ? GIT_BRANCH.split('/')[1..-1].join('/') : GIT_BRANCH}"
        branch = "${env.BRANCH_NAME}"
        registry = "gangzdh/doproject2"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'    
               
                echo 'env BRANCH_NAME: ' + env.BRANCH_NAME
                echo 'branch: ' + branch
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                   try{
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                          }
                    } catch(Exception e) {
                        echo 'Exception in Build Docker Image stage: ' + e.toString()
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    try{
                    //docker.withRegistry('https://registry.hub.docker.com', 'gangzdhlogincred') {
                        docker.withRegistry('', 'gangzdhlogincred') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                        }
                    } catch(Exception e) {
                        echo 'Exception in Push Docker Image stage: ' + e.toString()
                    }                    
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                script{
              try{
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true                    
                    )
                 } catch(Exception e) {
                  echo 'error in CanaryDeploy stage'
                  echo 'Exception in CanaryDeploy stage: ' + e.toString()
                 }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                //try{
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
                    //} catch(Exception e) {
                    //    echo 'Exception in DeployToProduction stage: ' + e.toString()
                    //}
            }
        }
    }
}
