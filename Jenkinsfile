def commit_id
pipeline {
    agent any
    environment {
		DOCKERHUB_CREDENTIALS=credentials('zbidis')
	}
    stages {
        stage('preparation') {
            steps {
                checkout scm
                sh "git rev-parse --short HEAD > .git/commit-id"
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }
        stage ('build') {
            steps {
                echo 'building maven workload'
                sh "mvn clean install"
                echo 'build complete'
            }
        }

        stage ("image build") {
            steps {
                echo 'building docker image'
                sh "docker build -t szbidi/position-simulator:${commit_id} ."
                echo 'docker image built'
            }
        }
        
        stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}
        stage ('Image Push') {
            steps {
                
                sh "docker push szbidi/position-simulator:${commit_id}"
            }
        }
        stage('deploy') {
            steps {
                sh "sed -i -r 's|richardchesterwood/k8s-fleetman-position-simulator:release2|szbidi/position-simulator:${commit_id}|' workloads.yaml"
                sh 'kubectl apply -f workloads.yaml'
            }
        }
    }
}
