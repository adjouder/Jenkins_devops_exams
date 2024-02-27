pipeline {
    environment { // Declaration of environment variables
        DOCKER_HUB_USERNAME = "adjouder" // replace this with your docker-id
        DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
        DOCKER_COMPOSE_FILE = "docker-compose.yml"
    }
    agent any // Jenkins will be able to select all available agents

    stages {
        stage('Docker Build') {
            steps {
                script {
                    // Construction et publication des images
                    sh "docker-compose -f ${DOCKER_COMPOSE_FILE} up -d"
                }
            }
        }

    stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
        steps {
                script {
                sh '''
                curl localhost
                '''
                }
        }
    }

    stage('Docker Push') {
        steps {
            script {
                // Authentification Docker
                sh "docker login -u ${DOCKER_HUB_USERNAME} -p ${DOCKER_HUB_PASS}"
                
                // Construction et publication des images
                sh "docker-compose -f ${DOCKER_COMPOSE_FILE} push"
            }
        }
    }

    stage('Deploiement en dev'){
            environment
            {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp movie-service/movieapp/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app movieapp --values=values.yml --namespace dev
                '''
                }
            }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp cast-service/castapp/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app castapp --values=values.yml --namespace dev
                '''
                }
            }

            }

    stage('Deploiement en QA'){
            environment
            {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
           steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp movie-service/movieapp/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app movieapp --values=values.yml --namespace qa
                '''
                }
            }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp cast-service/castapp/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app castapp --values=values.yml --namespace qa
                '''
                }
            }


            }

        stage('Deploiement en staging'){
            environment
                {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp movie-service/movieapp/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app movieapp --values=values.yml --namespace staging
                '''
                }
            }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp cast-service/castapp/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app castapp --values=values.yml --namespace staging
                '''
                }
            }

        }

        stage('Deploiement en prod'){
            environment
            {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
                steps {
                // Create an Approval Button with a timeout of 15minutes.
                // this require a manuel validation in order to deploy on production environment
                        timeout(time: 15, unit: "MINUTES") {
                            input message: 'Do you want to deploy in production ?', ok: 'Yes'
                        }

                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp cast-service/castapp/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app castapp --values=values.yml --namespace prod
                    '''
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp movie-service/movieapp/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app movieapp --values=values.yml --namespace prod
                    '''
                    }
                }
        }
            
    }
}