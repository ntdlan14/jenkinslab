pipeline {
    agent {
        label 'slave'}
    environment{
        DOCKER_IMAGE = "ntdl14/nginx"
        // DOCKER_USERNAME= "ntdl14"
        // DOCKER_PASSWORD= "Duyl@nlx92"
    }
    stages {
        stage("Build"){
            options {
                timeout(time: 10, unit: 'MINUTES')
            }
            environment {
                DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0,7)}"
            }
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . 
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    docker image ls | grep ${DOCKER_IMAGE}'''
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }

                //clean to save disk
                sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh "docker image rm ${DOCKER_IMAGE}:latest"
            }
        }
        stage("Deploy"){
            options {
                timeout(time: 10, unit: 'MINUTES')
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    ansiblePlaybook(
                        credentialsId: 'privatekey',
                        playbook: 'playbook.yaml',
                        inventory: 'inven.ini',
                        become: 'yes',
                        extraVars: [
                            DOCKER_USERNAME: "${DOCKER_USERNAME}",  
                            DOCKER_PASSWORD: "${DOCKER_PASSWORD}" 
                        ]
                    )
                }
                // sh 'ansible all -m ping -i inven.ini'
            }
        }
    }
    post {
        success {
            echo "SUCCESSFULL"
        }
        failure {
            echo "FAILED"
        }
    }
}