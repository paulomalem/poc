pipeline {

  agent any      

	environment {
        //Others Credentials
		DOCKERHUB_CREDENTIALS=credentials('dockerhub-pauloalem-cred')

        //Application Credentials
        REDIS_PASSWORD=credentials('redis-password-cred')
		SERVICO_2_BASE_URL=credentials('servico-dois-base-url-cred')
		SENHA_POSTGRES=credentials('senha-postgress-cred')
        PRIMEIRA_SECRET=credentials('PRIMEIRA_SECRET')

        //Kubernetes Credentials
        DEV_KUBECONFIG = credentials('dev-kubeconfig-cred')
        HML_KUBECONFIG = credentials('dev-kubeconfig-cred')
        PRD_KUBECONFIG = credentials('dev-kubeconfig-cred')
	}
	stages {
		stage('Build') {
            // agent { label 'linux' }
			steps {
                sh """#!/bin/bash +x
                    docker build -t first .
                """
            }
		}
        stage('Tagging') {
            // agent { label 'linux' }
			steps {
				sh """#!/bin/bash +x
                    docker tag first paulomalem/first:$BUILD_NUMBER
                """
			}
		}
		stage('Push Image (DokerHub') {
            // agent { label 'linux' }
			steps {
				sh """#!/bin/bash +x
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
        stage('Approve GC?') {
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
                    script {
                        sh 'mkdir ~/.kube'
                        sh 'cat $KUBECRED > ~/.kube/config'
                        sh 'kubectl get nodes'
                        //Capturando antiga imagem
                        env.PREVIOUS_IMAGE = sh( script: 'kubectl -n poc get deployments primeiro-servico -o=jsonpath="{$.spec.template.spec.containers[:1].image}"',
                             returnStdout: true).trim()
                        echo "PREVIOUS_IMAGE: ${env.PREVIOUS_IMAGE}"
                        sh 'for yml in ymls/* ; do envsubst < $yml | kubectl apply -f - ; done'
                        sh 'kubectl -n poc set image deployment/primeiro-servico primeiro-servico=paulomalem/first:$BUILD_NUMBER'
                        sh 'kubectl -n poc rollout status deployment.v1.apps/primeiro-servico'
                    }
                        
                }
            }
        }
        stage('Validação') {
            when {
                branch 'main'
            }
            steps {
                timeout(30) {
                    script {
                        env.flagError = "false"
                        try {
                            input(message: 'Valide esse job dentro dos proximos 30 minutos, caso contrário o rollback será iniciado.', ok: 'Proceed')

                        }catch(e){
                            println "Abortado ou timeout estourando, iniciando rollback...."
                            env.flagError = "true"        
                        }
                    }
                }
            }
        }
        stage("Rollback"){
            when{
                expression { env.flagError == "true" }
            }
            steps{
                withCredentials([file(credentialsId: 'dev-kubeconfig-cred', variable: 'KUBECRED')]) {
                    script {
                        sh 'rm -rf ~/.kube'
                        sh 'mkdir ~/.kube'
                        sh 'cat $KUBECRED > ~/.kube/config'
                        sh 'kubectl get nodes'
                        sh 'for yml in ymls/* ; do envsubst < $yml | kubectl apply -f - ; done'
                        sh "kubectl -n poc set image deployment/primeiro-servico primeiro-servico=${env.PREVIOUS_IMAGE}"
                        sh 'kubectl -n poc rollout status deployment.v1.apps/primeiro-servico'
                    }
                }
            }
        }
	}
	post {
		always {
			sh 'rm -rf ~/.kube'
		}
	}

}