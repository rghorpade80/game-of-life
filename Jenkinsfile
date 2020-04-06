pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "rghorpade80/gameoflife"
        //GIT_COMMIT jenkins variable to tag docker images (first 7 chra of git commit hash)
        SHORT_COMMIT = "${GIT_COMMIT[0..7]}"
    }
	
	options {
				buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '2')
			}

	stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
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
                        app.push("${SHORT_COMMIT}")
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
                input 'Deploy to Production?'
                milestone(1)
                
               
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'game-of-life.yml',
                    enableConfigSubstitution: true
                )
            }
        }