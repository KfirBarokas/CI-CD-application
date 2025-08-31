
pipeline {
    agent { 
	docker {
	    // because we are using an agent, the agent's container needs aws-cli installed
	    image 'jenkins-docker-aws-agent:latest'
	}
    }

    stages {

        stage('Build image') {
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
		steps{
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
        }

        stage('Deploy to production') {
            steps {
		script{
			if (env.BRANCH_NAME == 'main'){
				// install docker 
				sh '''ssh -i ~/kfir-key.pem ec2-user@13.221.96.99'''
				sh '''aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 992382545251.dkr.ecr.us-east-1.amazonaws.com
				&& docker pull 992382545251.dkr.ecr.us-east-1.amazonaws.com/kfirapp:latest
				&& docker run -it --rm kfirapp:latest
				'''			
			}
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
