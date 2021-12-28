pipeline{

	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub-pauloalem-cred')
		REDIS_PASSWORD=credentials('redis-password-cred')
		SERVICO_2_BASE_URL=credentials('servico-dois-base-url-cred')
		SENHA_POSTGRES=credentials('senha-postgress-cred')
        DEV_KUBECONFIG = credentials('dev-kubeconfig-cred')
        // PRD_KUBECONFIG = credentials('prd-kubeconfig-cred')
	}

	stages {

		// stage('Build') {
		// 	steps {
		// 		sh """
        //             docker build -t paulomalem/first:$BUILD_NUMBER .
        //         """
        //         // sh """
        //         //     N/A :D
        //         // """
        //     }
		// }
		// stage('Login') {
		// 	steps {
		// 		sh """
        //             echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
        //         """
        //         // sh """
        //         //     N/A :D
        //         // """                
		// 	}
		// }
		// stage('Push') {
		// 	steps {
		// 		sh """
        //             docker push 'paulomalem/first:$BUILD_NUMBER'
        //         """
        //         // sh """
        //         //     N/A :D
        //         // """
		// 	}
		// }
		stage('Deploy Image') {
			steps {
                // echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                sh """
                    docker build -t first .
                    docker tag first paulomalem/first:$BUILD_NUMBER
                    docker logout
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push paulomalem/first:$BUILD_NUMBER || exit 0
                 """
			}
		}
        stage('Deploy K8S (Desenvolvimento)') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([file(credentialsId: 'dev-kubeconfig-cred', variable: 'KUBECRED')]) {
                    sh 'mkdir ~/.kube'
                    sh 'cat $KUBECRED > ~/.kube/config'
                    sh 'kubectl get nodes'
                    sh 'for yml in ymls/* ; do envsubst < $yml | kubectl apply -f - ; done'
                    sh 'kubectl -n poc set image deployment/primeiro-servico/first:$BUILD_NUMBER'
                    sh 'kubectl -n poc rollout status deployment.v1.apps/primeiro-servico'
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