pipeline {
    agent any
    environment {
        PROJECT_ID = ""
        CLUSTER_NAME = ""
        LOCATION = ""
        CREDENTIALS_ID = ""
        DOCKER
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                script {
                    docker.withRegistry(env.DOCKER_REGISTRY, env.DOCKER_CREDENTIAL) {
                        def customImage = docker.build(env.DOCKER_IMAGE)
                        customImage.push()
                    }
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
                    def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'POST', requestBody: app, url: "${env.APISERVER_URL}/v1/namespaces/${env.APPLICATION_NAMESPACE}/applications/${env.APPLICATION_NAME}"
                    println('Status: '+response.status)
                    println('Response: '+response.content)
                }
            }
        }
    }
}
