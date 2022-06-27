def commit_id
pipeline {
    agent any
    
    environment {
        imageName = "myphpapp"
        registryCredentials = "nexus"
        registry = "http://localhost:8081/#browse/browse:maven-public"
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
                sh "docker build -t szbidi/position-simulator:${commit_id} ."
                echo 'docker image built'
            }
        }
        
       
        stage ('Image Push') {
            steps {
                
                sh "docker push szbidi/position-simulator:${commit_id}"
            }
        }
        
         stage ('MVN Install') {
            steps {
                echo "Maven cpying all jar to our local repo";
                sh 'mvn install -Dmaven.test.skip=true';
            }
        
         stage ('MVN DEPLOY') {
            steps {
                echo "Maven cpying all jar to our nexus remote repo";
                sh 'mvn -Dmaven.test.skip=true deploy:deploy-file -DgroupId=tn.esprit -DartifactId=timesheet-devops -Dversion=1.0 -DgeneratePom=true -Dpackaging=jar -DrepositoryId=deploymentRepo -Durl=http://localhost:8081/#browse/browse:maven-releases -Dfile=target/timesheet-devops-1.0.jar';

            }
        }
        
        stage('deploy') {
            steps {
                sh "sed -i -r 's|richardchesterwood/k8s-fleetman-position-simulator:release2|szbidi/position-simulator:${commit_id}|' workloads.yaml"
                sh 'kubectl apply -f workloads.yaml'
            }
        }
    
}
