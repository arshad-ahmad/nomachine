pipeline {
    agent any
    environment {
        GIT_BRANCH = 'main'
        GIT_URL = 'https://github.com/arshad-ahmad/vela-jenkins.git'
        DOCKER_REGISTRY = 'https://registry.hub.docker.com'
        DOCKER_CREDENTIAL = 'DockerHubCredential'
        DOCKER_IMAGE = 'arshad1914/kubevela-demo-cicd-app'
        APISERVER_URL = 'https://35.185.162.243'
        APPLICATION_YAML = 'app.yaml'
        APPLICATION_NAMESPACE = 'kubevela-demo-namespace'
        APPLICATION_NAME = 'cicd-demo-app'
    }
    stages {
        stage('Prepare') {
            steps {
                script {
                    def checkout = git branch: env.GIT_BRANCH, url: env.GIT_URL
                    env.GIT_COMMIT = checkout.GIT_COMMIT
                    env.GIT_BRANCH = checkout.GIT_BRANCH
                    echo "env.GIT_BRANCH=${env.GIT_BRANCH},env.GIT_COMMIT=${env.GIT_COMMIT}"
                }
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
                    def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'POST', requestBody: app, url: "https://35.185.162.243/v1/namespaces/kubevela-demo-namespace/applications/cicd-demo-app"
                    println('Status: '+response.status)
                    println('Response: '+response.content)
                }
            }
        }
    }
    post {
        success {
            setBuildStatus("Deploy success", "SUCCESS");
        }
        failure {
            setBuildStatus("Deploy failed", "FAILURE");
        }
    }
}
