
pipeline {
    agent { 
	docker {
	    image 'docker:latest'
	}
    }

    //environment {
    //    GIT_CREDENTIALS_ID = 'GITHUB_KEY'
    //    GIT_REPO_URL = 'https://github.com/KfirBarokas/CI-CD-application.git'
    //}

    stages {

        stage('Build image') {
        when {
		branch 'dev'
	} 
	steps {
                //sh 'pip install -r requirements.txt'
		sh '''docker build -t kfirapp . 
		docker tag kfirapp:latest 992382545251.dkr.ecr.us-east-1.amazonaws.com/kfirapp:dev'''
		
            }
        }

        stage('Run tests') {
	    agent { 
		docker {
		    image 'docker:latest'
		}
	    }
            steps {
		sh 'docker run -d kfirapp sh -c "python3 -m unittest discover -s tests -v"'
		echo 'running tests'
            }
        }

        stage('Push to ecr') {
            when {
		    branch 'dev'
	    }
	    steps {
		// because we are using an agent, the agent's container needs aws-cli installed
		sh '''https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
		&& unzip awscliv2.zip
		&& ./aws/install 
		&& aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 992382545251.dkr.ecr.us-east-1.amazonaws.com 
		&& docker push 992382545251.dkr.ecr.us-east-1.amazonaws.com/kfirapp:dev'''
		//script{
		//	docker.withRegistry("https://992382545251.dkr.ecr.us-east-1.amazonaws.com", "AWS_INSTANCE_ROLE") {
		//		docker.image("kfirapp:dev").push()
		//	}
		//}
		echo 'deployin!!!'
            }
        }

        stage('Push changes (if any)') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS_ID}", usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh '''
                        git config user.name "jenkins"
                        git config user.email "jenkins@example.com"

                        if ! git diff --quiet; then
                            git add .
                            git commit -m "Auto commit by Jenkins after successful tests"
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@${GIT_REPO_URL#https://} HEAD:${BRANCH}
                        else
                            echo "No changes to push."
                        fi
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed – no push or deployment executed.'
        }
    }
}
