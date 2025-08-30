
pipeline {
    agent { 
	docker {
	    // because we are using an agent, the agent's container needs aws-cli installed
	    image 'jenkins-docker-aws-agent:latest'
	}
    }

    stages {

        stage('Build image') {
        when {
		branch 'dev'
	} 
	steps {
		script{
			// tagging according to branch 
			sh 'docker build -t kfirapp . '
			if (env.BRANCH_NAME == 'dev'){
				sh 'docker tag kfirapp:latest 992382545251.dkr.ecr.us-east-1.amazonaws.com/kfirapp:dev'
			}
			else if (env.BRANCH_NAME == 'main'){
				sh 'docker tag kfirapp:latest 992382545251.dkr.ecr.us-east-1.amazonaws.com/kfirapp:latest'
			}
		}
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
		script {
			sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 992382545251.dkr.ecr.us-east-1.amazonaws.com'
                        if (env.BRANCH_NAME == 'dev'){
                                sh 'docker push 992382545251.dkr.ecr.us-east-1.amazonaws.com/kfirapp:dev'
                        }
                        else if (env.BRANCH_NAME == 'main'){
                                sh 'docker push 992382545251.dkr.ecr.us-east-1.amazonaws.com/kfirapp:latest'
                        }
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
