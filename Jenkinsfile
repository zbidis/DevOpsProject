def commit_id
pipeline {
    agent any
    
    environment {
        imageName = "position-simulator:${commit_id}"
        registryCredentials = "nexusadmin"
        registry = "127.0.0.1:9001/docker-hosted"
        dockerImage = ''
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
                sh "docker build -t 127.0.0.1:9001/docker-hosted/position-simulator:${commit_id} ."
                echo 'docker image built'
            }
        }
        
       
       // stage ('Image Push in Dockerhub') {
          //  steps {
                
                //sh "docker push szbidi/position-simulator:${commit_id}"
            //}
       // }
        
        stage ('Image Push in Nexus') {
            steps {
             
                //sh "docker push 127.0.0.1:9001/docker-hosted/position-simulator:${commit_id}"
                docker.withRegistry( '', registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
            }
        }
        
       
        
        
        stage('deploy') {
            steps {
                sh "sed -i -r 's|richardchesterwood/k8s-fleetman-position-simulator:release2|szbidi/position-simulator:${commit_id}|' workloads.yaml"
                sh 'kubectl apply -f workloads.yaml'
            }
        }
          
       }
        stage('Remove Unused docker image') {
            steps{
                sh "docker rmi $imagename:$BUILD_NUMBER"
                sh "docker rmi $imagename:latest"

      }
     }
}
