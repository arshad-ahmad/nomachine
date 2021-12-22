pipeline {
    agent any
    environment {
        GIT_BRANCH = 'main'
        GIT_URL = 'https://github.com/arshad-ahmad/vela-jenkins.git'
        DOCKER_REGISTRY = 'https://registry.hub.docker.com'
        DOCKER_CREDENTIAL = 'DockerHubCredential'
        DOCKER_IMAGE = 'arshad1914/kubevela-demo-cicd-app'
        APISERVER_URL = 'https://34.81.131.154'
        APPLICATION_YAML = 'app.yaml'
        APPLICATION_NAMESPACE = 'kubevela-demo-namespace'
    }
    stages {
        stage('CheckOut~') {
            steps {
                    checkout scm
		}
	}
        stage('Build') {
            steps {
                script {
                  withCredentials( [string(credentialsId: 'DockerHubCredential', variable: 'DockerHubCredential')]) {
                    sh "docker login -u arshad1914 -p ${DockerHubCredential}"
                  }
                        def customImage = docker.build(env.DOCKER_IMAGE)
                        customImage.push()
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'wget -q "https://github.com/mikefarah/yq/releases/download/v4.12.1/yq_linux_amd64"'
                sh 'chmod +x yq_linux_amd64'
                script {
                    def app = sh (
                        script: "./yq_linux_amd64 eval -o=json '.spec' ${env.APPLICATION_YAML} | sed -e 's/GIT_COMMIT/$GIT_COMMIT/g'",
                        returnStdout: true
                    )
                    echo "app: ${app}"
                    def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'POST', requestBody: app, url: "http://http://34.133.183.85:32680/v1/namespaces/kubevela-demo/applications/cicd-demo-app"
                    println('Status: '+response.status)
                    println('Response: '+response.content)
                }
            }
        }
    }
}
