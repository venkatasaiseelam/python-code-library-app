pipeline {
    agent {
        label 'docker'
    }
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "devops1117/pythonlib"
    }
    
    stages {
        stage('Code checkout') {
            steps {
                git 'https://github.com/venkatasaiseelam/python-code-library-app.git'
            }
        }
        stage("Code Q.A.") {
            environment {
                scannerHome = tool 'SonarSC'
            }
            steps{
                withSonarQubeEnv('mysonar') {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=library-app"
                }
            }
        }
        stage('Image Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:auth_v${IMAGE_TAG} auth"
                sh "docker build -t ${IMAGE_NAME}:book_v${IMAGE_TAG} book"
                sh "docker build -t ${IMAGE_NAME}:borrow_v${IMAGE_TAG} borrow"
                sh "docker build -t ${IMAGE_NAME}:db_v${IMAGE_TAG} database"
                sh "docker build -t ${IMAGE_NAME}:frontend_v${IMAGE_TAG} ."
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh """
                trivy image ${IMAGE_NAME}:auth_v${IMAGE_TAG} >> auth_report.txt
                trivy image ${IMAGE_NAME}:book_v${IMAGE_TAG} >> book_report.txt
                trivy image ${IMAGE_NAME}:borrow_v${IMAGE_TAG} >> borrow_report.txt
                trivy image ${IMAGE_NAME}:db_v${IMAGE_TAG} >> db_report.txt
                trivy image ${IMAGE_NAME}:frontend_v${IMAGE_TAG} >> frontend_report.txt
                """
            }
        }
        stage('registry') {
            steps{
                script {
                    withDockerRegistry(credentialsId: 'ff8dee13-be86-4c97-8c1b-52e7603272b4') {
                        sh """
                        docker push ${IMAGE_NAME}:auth_v${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:book_v${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:borrow_v${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:db_v${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:frontend_v${IMAGE_TAG}
                        """
                    }
                }
            }
        }
        stage('update manifest files') {
            steps {
                script {
                    withCredentials([gitUsernamePassword(credentialsId: '8670ae89-1b23-4c85-8bd3-9a431f033de8', gitToolName: 'Default')]) {
                        dir('manifests') {
                            sh """
                            yq -i '.spec.template.spec.containers[0].image = "${IMAGE_NAME}:auth_v${IMAGE_TAG}"' auth_deployment.yaml
                            yq -i '.spec.template.spec.containers[0].image = "${IMAGE_NAME}:book_v${IMAGE_TAG}"' book_deployment.yaml
                            yq -i '.spec.template.spec.containers[0].image = "${IMAGE_NAME}:borrow_v${IMAGE_TAG}"' borrow-deployment.yaml
                            yq -i '.spec.template.spec.containers[0].image = "${IMAGE_NAME}:db_v${IMAGE_TAG}"' db-deployment.yaml
                            yq -i '.spec.template.spec.containers[0].image = "${IMAGE_NAME}:frontend_v${IMAGE_TAG}"' frontend-deployment.yaml
                            """
                            
                            sh """
                            git config user.email "admin@devopsorg.test"
                            git config user.name "Admin"
                            
                            git add .
                            git commit -m "Updated images versions to ${IMAGE_TAG}" || true
                            git push origin master
                            """
                        }
                    }
                }
            }
        }
    }
}
