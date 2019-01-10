pipeline {
    agent {
        node {
            label 'slave-03'
        }
    }

    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "godfather/train-schedule"
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
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                //input 'Deploy to Production?'
                //milestone(1)
                sh 'kubectl get pods'
                //kubernetesDeploy(
                //    kubeconfigId: 'kubeconfig',
                //    configs: 'train-schedule-kube.yml',
                //    enableConfigSubstitution: true
                //)

                kubernetesDeploy {
                    context {
                    // The path patterns for the Kubernetes configurations you want to deploy, in the form of Ant glob syntax.
                    configs('train-schedule-kube.yml')
                     // Choose how to get the <a href="https://kubernetes.io/docs/concepts/cluster-administration/authenticate-across-clusters-kubeconfig/"> kubeconfig file to authenticate with the Kubernetes cluster management endpoint. 3 options are supported: Authenticate with Kube config file - Get the kubeconfig file from the workspace path. Deprecated.
                    //credentialsType('Fill credentials details directly')
                    //dockerCredentials {}
                    // Substitute variables (in the form $VARIABLE or ${VARIABLE}) in the configuration with values from Jenkins environment variables.
                    enableConfigSubstitution(true)
                    //kubeConfig {}
                    kubeconfigId('kubeconfig')
                    // The secret name that you can use in the Kubernetes Deployment configuration for the imagePullSecrets entry.
                    //secretName(String value)
                    // The Kubernetes namespace in which the secrets will be created with the credentials configured below.
                    //secretNamespace(String value)
                    //ssh {}
                    //textCredentials {}
                }
                }
            }
        }
    }
}
