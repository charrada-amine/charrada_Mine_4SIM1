pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        DOCKERHUB_USER = 'slimane69'
        IMAGE_NAME = 'devops-mohamed-slimane'
        K8S_NAMESPACE = 'devops'
        K8S_MANIFEST_DIR = 'k8s'
        K8S_DEPLOYMENT_NAME = 'student-management-app'
        KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/MedSlimane/devops-mohamed-slimane-4sim1'
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        // stage('SonarQube Analysis') {
        //     steps {
        //         withSonarQubeEnv('xsonar') { // uses the global SonarQube server config
        //             sh 'mvn sonar:sonar'
        //         }
        //     }
        // }

        stage('Maven Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'USER',
                                                 passwordVariable: 'PASS')]) {
                    sh """
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                    """
                                                 }
            }
        }

        stage('Docker Push') {
            steps {
                sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIAL_ID}", variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl apply -f ${K8S_MANIFEST_DIR}/
                        kubectl -n ${K8S_NAMESPACE} rollout restart deployment/${K8S_DEPLOYMENT_NAME}
                    """
                }
            }
        }

        stage('Verify Rollout') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIAL_ID}", variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl -n ${K8S_NAMESPACE} rollout status deployment/${K8S_DEPLOYMENT_NAME} --timeout=180s
                        kubectl -n ${K8S_NAMESPACE} get pods -o wide
                        kubectl -n ${K8S_NAMESPACE} get svc
                    """
                }
            }
        }
    }
}
