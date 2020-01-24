pipeline {
    options {
        parallelsAlwaysFailFast()
    }
    environment {
        //registryCredential = 'harbor-lol-jenk-robot'
		registryCredential = 'registry-gulla-xyz'
        DOCKER_IMAGE = sh 'echo ${JOB_NAME} | sed "s/gitea-larkspur//g" | sed "s/master//g"'
        registry = "" /*be sure to include trailing slash*/
        docker_image_name = "bgulla/assistant-relay"
        docker_img =  "${registry}${docker_image_name}"

        push_to_local_registry = false
        //local_registry = "harbor.lol"
		local_registry = "reg.gulla.xyz"
        local_registry_namespace = "dev"
    }
    //  agent any
    agent {
        label 'arm'
    }
    stages {
        stage ('build multi-arch container'){
            parallel{
                stage('Build AMD64 Docker and Push to Registries') {
                agent {
                    label 'amd64'
                }
                steps {
                    script {
                        sh 'whoami'
                        sh 'env'
                        docker.withRegistry( '', 'dockerhub-creds' ) {
                            dockerImage = docker.build docker_img + ":amd64"
                            dockerImage.push()
                        }
                    //  if (true){
                            docker.withRegistry( 'https://${local_registry}', 'registry-gulla-xyz' ) {
                                local_registry_docker_image = "${local_registry_namespace}/${docker_img}"
                                sh "echo ${local_registry_docker_image}" 
                                dockerImage = docker.build local_registry_docker_image + ":amd64"
                                dockerImage.push()
                            }
                    //   }
                    }
                }
            }

            stage('Build ARMv7 Docker and Push to Registries') {
                agent {
                    label 'arm'
                }
                steps {
                    script {
                        sh 'whoami'
                        sh 'env'
                        docker.withRegistry( '', 'dockerhub-creds' ) {
                            dockerImage = docker.build docker_img + ":armv7"
                            dockerImage.push()
                        }
                        //if (true){
                            docker.withRegistry( 'https://${local_registry}', 'registry-gulla-xyz' ) {
                                local_registry_docker_image = "${local_registry}/${local_registry_namespace}/${docker_img}"
                                dockerImage = docker.build local_registry_docker_image + ":armv7"
                                dockerImage.push()
                            }
                    //  }
                    }
                }
            }


            stage('Build ARM64 Docker and Push to Registries') {
                agent {
                    label 'arm64'
                }
                steps {
                    script {
                        sh 'whoami'
                        sh 'env'
                        docker.withRegistry( '', 'dockerhub-creds' ) {
                            dockerImage = docker.build docker_img + ":arm64"
                            dockerImage.push()
                        }
                        if (true){
                            docker.withRegistry( 'https://${local_registry}', 'registry-gulla-xyz' ) {
                                local_registry_docker_image = "${local_registry}/${local_registry_namespace}/${docker_img}"
                                dockerImage = docker.build local_registry_docker_image + ":arm64"
                                dockerImage.push()
                            }
                        }
                    }
                }
            }
        }
        }
        stage('Build Docker Manifest and push') {
            agent {
                label 'amd64' /*amd only*/
            }
            steps {
                script {
                    sh 'whoami'
                    sh 'env'
                    docker.withRegistry( '', 'dockerhub-creds' ) {
                        sh 'docker version'
                        sh "docker manifest create ${docker_img}:latest ${docker_img}:armv7 ${docker_img}:amd64 ${docker_img}:armv7 ${docker_img}:arm64"
                        sh "docker manifest push ${docker_img}:latest"
                    }
                }
            }
        }
    }
}
