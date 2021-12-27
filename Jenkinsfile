pipeline{

	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub-pauloalem-cred')
		REDIS_PASSWORD=credentials('redis-password-cred')
		SERVICO_2_BASE_URL=credentials('servico-dois-base-url-cred')
		SENHA_POSTGRES=credentials('senha-postgress-cred')
        // DEV_KUBECONFIG = credentials('dev-kubeconfig-cred')
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
                    docker logout
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker build -t paulomalem/first:$BUILD_NUMBER .
                    docker push 'paulomalem/first:$BUILD_NUMBER' || :
                // """
                //     kubectl --kubeconfig $DEV_KUBECONFIG get pods
                //     for yml in k8s/development/* ; do envsubst < $yml | kubectl apply -f - ; done
                //     cat k8s/development/secret.yml
                //     kubectl -n poc set image deployment/paulomalem/first:$BUILD_NUMBER
                //     kubectl -n poc rollout status deployment.v1.apps/primeiro-servico
			}
		}
        stage('Deploy K8S (Desenvolvimento)') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying'
            }
        }        
	}
	// post {
	// 	always {
	// 		sh 'docker logout'
	// 	}
	// }

}