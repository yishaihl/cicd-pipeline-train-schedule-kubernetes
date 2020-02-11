pipeline {
    agent { label 'k8s-jobs' }
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "yishaihl32/train-schedule"
        PROJECT_ID = "optimal-leon"
        CLUSTER_NAME = "gke-jenkins-cluster"
        LOCATION = "us-central1-a"
        CREDENTIALS_ID = '4Optimal-leon'
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
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
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to GKE') {
            when {
                branch 'master'
            }
            steps{
                step([
                $class: 'KubernetesEngineBuilder',
                projectId: env.PROJECT_ID,
                clusterName: env.CLUSTER_NAME,
                location: env.LOCATION,
                manifestPattern: sh "sed -i 's/$DOCKER_IMAGE_NAME:$BUILD_NUMBER/"(DOCKER_IMAGE_NAME)":"${env.BUILD_NUMBER}"/g' train-schedule-kube.yml",
                credentialsId: env.CREDENTIALS_ID,
                verifyDeployments: true])
            }
        }
    }
}
