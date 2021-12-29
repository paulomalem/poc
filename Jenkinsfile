pipeline {

  agent any      

	environment {
        //Others Credentials
		DOCKERHUB_CREDENTIALS=credentials('dockerhub-pauloalem-cred')
		
        //Application Credentials
        REDIS_PASSWORD=credentials('redis-password-cred')
		SERVICO_2_BASE_URL=credentials('servico-dois-base-url-cred')
		SENHA_POSTGRES=credentials('senha-postgress-cred')
        
        //Kubernetes Credentials
        DEV_KUBECONFIG = credentials('dev-kubeconfig-cred')
        HML_KUBECONFIG = credentials('dev-kubeconfig-cred')
        PRD_KUBECONFIG = credentials('dev-kubeconfig-cred')
	}
	stages {
		stage('Build') {
            // agent { label 'linux' }
			steps {
				sh """
                    docker build -t first .
                """
            }
		}
        stage('Tagging') {
            // agent { label 'linux' }
			steps {
				sh """
                    docker tag first paulomalem/first:$BUILD_NUMBER
                """
			}
		}
		stage('Push Image (DokerHub') {
            // agent { label 'linux' }
			steps {
				sh """
                    docker logout
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push paulomalem/first:$BUILD_NUMBER || exit 0
                """
			}
		}
        stage('Deploy K8S (Desenvolvimento)') {
            // agent { label 'linux' }
            when {
                branch 'develop'
            }
            steps {
                withCredentials([file(credentialsId: 'dev-kubeconfig-cred', variable: 'KUBECRED')]) {
                    sh 'mkdir ~/.kube'
                    sh 'cat $KUBECRED > ~/.kube/config'
                    sh 'kubectl get nodes'
                    sh 'for yml in ymls/* ; do envsubst < $yml | kubectl apply -f - ; done'
                    sh 'kubectl -n poc set image deployment/primeiro-servico primeiro-servico=paulomalem/first:$BUILD_NUMBER'
                    sh 'kubectl -n poc rollout status deployment.v1.apps/primeiro-servico'
                }
            }
        }    
        stage('Deploy K8S (Homologação)') {
            // agent { label 'linux' }
            when {
                branch 'homolog'
            }
            steps {
                withCredentials([file(credentialsId: 'dev-kubeconfig-cred', variable: 'KUBECRED')]) {
                    sh 'mkdir ~/.kube'
                    sh 'cat $KUBECRED > ~/.kube/config'
                    sh 'kubectl get nodes'
                    sh 'for yml in ymls/* ; do envsubst < $yml | kubectl apply -f - ; done'
                    sh 'kubectl -n poc set image deployment/primeiro-servico primeiro-servico=paulomalem/first:$BUILD_NUMBER'
                    sh 'kubectl -n poc rollout status deployment.v1.apps/primeiro-servico'
                }
            }
        }
        stage('Deploy K8S (Homologação 02)') {
            // agent { label 'linux' }
            when {
                branch 'homolog'
            }
            steps {
                input message: "Deploy em Homologação 02?"
                withCredentials([file(credentialsId: 'dev-kubeconfig-cred', variable: 'KUBECRED')]) {
                    sh 'mkdir ~/.kube'
                    sh 'cat $KUBECRED > ~/.kube/config'
                    sh 'kubectl get nodes'
                    sh 'for yml in ymls/* ; do envsubst < $yml | kubectl apply -f - ; done'
                    sh 'kubectl -n poc set image deployment/primeiro-servico primeiro-servico=paulomalem/first:$BUILD_NUMBER'
                    sh 'kubectl -n poc rollout status deployment.v1.apps/primeiro-servico'
                }
            }
        }
        stage('Produção?') {
            agent none
            when {
                branch 'main'
            }           
            options {
                timeout(time: 1, unit: 'HOURS') 
            }
            steps {
                input message: "Deploy em Produção?"
            }
        }
        stage('Deploy K8S (Produção)') {
            // agent { label 'linux' }
            when {
                branch 'main'
            }
            steps {
                withCredentials([file(credentialsId: 'dev-kubeconfig-cred', variable: 'KUBECRED')]) {
                    sh 'mkdir ~/.kube'
                    sh 'cat $KUBECRED > ~/.kube/config'
                    sh 'kubectl get nodes'
                    sh 'for yml in ymls/* ; do envsubst < $yml | kubectl apply -f - ; done'
                    sh 'kubectl -n poc set image deployment/primeiro-servico primeiro-servico=paulomalem/first:$BUILD_NUMBER'
                    sh 'kubectl -n poc rollout status deployment.v1.apps/primeiro-servico'
                }
            }
        }
        stage('validate') {
            steps {
                timeout(30) {
                    script {
                        CHOICES = ["deploy", "rollback"];    
                            env.yourChoice = input  message: 'Please validate, this job will automatically ABORTED after 30 minutes even if no user input provided', ok : 'Proceed',id :'choice_id',
                                            parameters: [choice(choices: CHOICES, description: 'Do you want to deploy or to rollback?', name: 'CHOICE'),
                                                string(defaultValue: 'rollback', description: '', name: 'rollback value')]
                            } 

                }
            }
        }
            stage('Deploy') {
                when {
                    expression { env.yourChoice == 'deploy' }
                }
                steps {
                    echo 'deploy'
                }
            }
            stage('Rollback') {
                when {
                    expression { env.yourChoice == 'rollback' }
                }
                steps {
                    echo 'rollback'
                }
            }
	}
	post {
		always {
			sh 'rm -rf ~/.kube'
		}
	}

}