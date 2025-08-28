
pipeline {
    agent { 
	docker {
	    image 'docker:latest'
	}
    }

    environment {
        GIT_CREDENTIALS_ID = 'GITHUB_KEY'
        GIT_REPO_URL = 'https://github.com/KfirBarokas/CI-CD-application.git'
        BRANCH = 'main'
	//AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
    	//AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: "${GIT_REPO_URL}", branch: "${BRANCH}"
            }
        }

        stage('Install dependencies') {
            
	steps {
                //sh 'pip install -r requirements.txt'
		echo 'insatlling deps'
		script {
	            def app = docker.build('kfirapp')
        	    //app.run('--rm -d -p 5001:5000')
        	}
		
            }
        }

        stage('Run tests') {
	    agent { 
		docker {
		    image 'python:3.9-alpine'
		}
	    }
            steps {
                //sh 'pytest tests/test1.py'
		sh 'docker run -d kfirapp sh -c "python3 -m unittest discover -s tests -v"'
                //sh 'pytest tests/test2.py'
		echo 'running tests'
            }
        }

        stage('Deploy') {
            steps {
		script{
			docker.withRegistry("https://992382545251.dkr.ecr.us-east-1.amazonaws.com", "ecr:us-east-1:AWS_CREDS") {
				docker.image("kfirapp").push()
			}
		}
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



        stage('Deploy') {
            steps {
		  echo 'Deploying...'
//                sh '''
//                    docker build -t myapp:latest .
//                    docker run -d -p 80:8000 myapp:latest
//                '''
            }
        }

