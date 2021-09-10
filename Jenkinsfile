pipeline {
    agent any
    environment {
        registryCredential = 'MyDockerHubCreds'
		registry = "semigr/goappsrepo"
        GOCACHE = "/tmp"
    }
    stages {
        stage('Build') {
            agent { 
                docker { 
                    image 'golang' 
                }
            }
            steps {
                // Create our project directory.
                sh 'cd ${GOPATH}/src'
                sh 'mkdir -p ${GOPATH}/src/hello-world'
                // Copy all files in our Jenkins workspace to our project directory.                
                sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
				
                // Build the app.
                sh 'go build'               
            }     
        }
        stage('Test') {
            agent { 
                docker { 
                    image 'golang' 
                }
            }
            steps {                 
                // Create our project directory.
                sh 'cd ${GOPATH}/src'
                sh 'mkdir -p ${GOPATH}/src/hello-world'
                // Copy all files in our Jenkins workspace to our project directory.                
                sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
                // Remove cached test results.
                sh 'go clean -cache'
                // Run Unit Tests.
                sh 'go test ./... -v -short'            
            }
        }
        stage('Publish') {
            environment {
                registryCredential = 'MyDockerHubCreds'
            }
            steps{
                script {
                    def appimage = docker.build registry + ":$BUILD_NUMBER"
                    docker.withRegistry( '', registryCredential ) {
                        appimage.push()
                        appimage.push('latest')
                    }
                }
            }
        }
        stage ('Deploy') {
            steps {
                script{
                    def image_id = registry + ":$BUILD_NUMBER"
					sh "whoami"
					sh "sed -i s#JUSTIMAGEPLACEHOLDER#${image_id}#g deployment.yml"
					sh "cat deployment.yml"
					sh "echo PATH=$PATH"
					sh "echo SHELL=$SHELL"
					sh "/usr/local/bin/kubectl apply -f deployment.yml"
                    sh "/usr/local/bin/kubectl apply -f service.yml"
                }
            }
        }
    }
}