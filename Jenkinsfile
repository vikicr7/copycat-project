pipeline {
    agent any

    parameters {
        choice(
            name: 'action',
            choices: 'create\ndelete',
            description: 'Choose create/Destroy'
        )
        string(name: 'ImageName', description: "name of the docker build", defaultValue: 'javapp')
        string(name: 'ImageTag', description: "tag of the docker build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "DockerHub username", defaultValue: 'vikicr7')
    }

    environment {
        DOCKER_IMAGE = "${DockerHubUser}/${ImageName}:${ImageTag}"
    }

    tools {
        jdk   'JDK17'     // configured in Jenkins → JDK
        maven 'Maven-3'   // configured in Jenkins → Maven
    }

    stages {

        stage('Git Checkout') {
            when { expression { params.action == 'create' } }
            steps {
                git url: "https://github.com/vikicr7/copycat-project.git",
                    branch: 'main'
            }
        }

        stage('Unit Test maven') {
            when { expression { params.action == 'create' } }
            steps {
                sh 'mvn test'
            }
        }

        stage('Integration Test maven') {
            when { expression { params.action == 'create' } }
            steps {
                sh 'mvn verify'
            }
        }

        stage('Static code analysis: Sonarqube') {
            when { expression { params.action == 'create' } }
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=copycat-project \
                          -Dsonar.projectName=copycat-project
                    '''
                }
            }
        }

        stage('Quality Gate Status Check : Sonarqube') {
            when { expression { params.action == 'create' } }
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Maven Build : maven') {
            when { expression { params.action == 'create' } }
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Image Build') {
            when { expression { params.action == 'create' } }
            steps {
                sh """
                    echo "Building Docker image: ${DOCKER_IMAGE}"
                    docker build -t ${DOCKER_IMAGE} .
                """
            }
        }

        stage('Docker Image Scan: trivy') {
            when { expression { params.action == 'create' } }
            steps {
                sh """
                    echo "Scanning Docker image with Trivy: ${DOCKER_IMAGE}"
                    trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE}
                """
            }
        }

        stage('Docker Image Push : DockerHub') {
            when { expression { params.action == 'create' } }
            steps {
                sh """
                    echo "Pushing Docker image: ${DOCKER_IMAGE}"
                    docker push ${DOCKER_IMAGE}
                """
            }
        }

        stage('Docker Image Cleanup : DockerHub') {
            when { expression { params.action == 'create' } }
            steps {
                sh """
                    echo "Cleaning up local Docker image: ${DOCKER_IMAGE}"
                    docker rmi ${DOCKER_IMAGE} || true
                """
            }
        }
    }
}
