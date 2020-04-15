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
                sh 'mvn clean deploy'
            }
        }
		
	stage('nexusArtifactUpload') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'gameoflife', classifier: '', file: 'gameoflife-web/target/gameoflife.war', type: '.war']], credentialsId: 'Nexus_credentials', groupId: 'com.wakaleo.gameoflife', nexusUrl: '13.58.106.100:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots/', version: '1.0-SNAPSHOT'
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
			    
			/*Delete images to free space on jenkins server */
			sh 'docker image prune -a --force'
			
			    
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
	}
}
