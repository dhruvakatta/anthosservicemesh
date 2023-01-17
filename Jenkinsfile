pipeline {
    agent {
        label "jenkins-slave"
    }
    environment {
        NEXUS_USERNAME = credentials("nexus_username")
        NEXUS_PASSWORD = credentials("nexus_password")
        KUBECONFIG = credentials("kubeconfig")
        CHART_NAME = "my-spring-boot-app"
        CHART_VERSION = "1.0.0"
    }
    options {
        skipDefaultCheckout true
        gitLabConnection('github')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/<username>/<repository-name>.git']]
                ])
            }
        }
        stage('Install dependencies') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Dockerize') {
            steps {
                sh 'docker build --tag my-spring-boot-app:${env.BUILD_TAG} .'
            }
        }
        stage('Sign') {
            steps {
                sh 'docker run --rm -v "$PWD:/workdir" openpgp/sign -u "Developer <dev@example.com>" -s -o /workdir/image.tar /workdir/my-spring-boot-app:${env.BUILD_TAG}'
                sh 'docker load -i /workdir/image.tar'
            }
        }
        stage('Push to Nexus') {
            steps {
                sh "docker login -u \$NEXUS_USERNAME -p \$NEXUS_PASSWORD nexus.example.com"
                sh "docker push nexus.example.com/my-spring-boot-app:${env.BUILD_TAG}"
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "$KUBECONFIG", variable: 'kubeconfig')]) {
                    sh '''
                    export KUBECONFIG=${kubeconfig}
                    helm upgrade --install --set image.repository=nexus.example.com/my-spring-boot-app,image.tag=${env.BUILD_TAG} ${env.CHART_NAME} charts/${env.CHART_NAME} --version ${env.CHART_VERSION}
                    '''
                }
            }
        }
    }
}