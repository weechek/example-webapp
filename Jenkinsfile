def builderImage
def productionImage
def ACCOUNT_REGISTRY_PREFIX
def GIT_COMMIT_HASH
def working_dir = '/var/jenkins_home/workspace/example-webapp_master'
pipeline {
    agent any
    stages {
        stage('Checkout Source Code and Logging Into Registry') {
            steps {
                echo 'Logging Into the Private ECR Registry'
                script {
                    GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
                    ACCOUNT_REGISTRY_PREFIX = "wctan"
                }
            }
        }

        stage('Make A Builder Image') {
            steps {
                echo 'Starting to build the project builder docker image'
                script {
					docker.withRegistry('', 'docker-hub-credentials') {
						builderImage = docker.build("${ACCOUNT_REGISTRY_PREFIX}/example-webapp-builder:${GIT_COMMIT_HASH}", "-f ./Dockerfile.builder .")
						builderImage.push()
						builderImage.push("${env.GIT_BRANCH}")
						builderImage.inside('-u root') {
							sh """
							   lein uberjar
							"""
						}
					}
                }
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'running unit tests in the builder image.'
                script {
                    builderImage.inside('-v /var/jenkins_home/workspace/example-webapp_master:/output -u root') {
                    sh """
                       lein test
                    """
                    }
                }
            }
        }

        stage('Build Production Image') {
            steps {
                echo 'Starting to build docker image'
                script {
					docker.withRegistry('', 'docker-hub-credentials') {
						productionImage = docker.build("${ACCOUNT_REGISTRY_PREFIX}/example-webapp:${GIT_COMMIT_HASH}")
						productionImage.push()
						productionImage.push("${env.GIT_BRANCH}")
					}
                }
            }
        }

 
        stage('Deploy to Production fixed server') {
            when {
                branch 'release'
            }
            steps {
                echo 'Deploying release to production'
                script {
                    productionImage.push("deploy")
                    sh """
                       aws ec2 reboot-instances --region us-east-1 --instance-ids i-0e438e2bf64427c9d
                    """
                }
            }
        }


        stage('Integration Tests') {
            when {
                branch 'release'
            }
            steps {
                echo 'Deploy to test environment and run integration tests'
                script {
                    TEST_ALB_LISTENER_ARN="arn:aws:elasticloadbalancing:us-east-1:089778365617:listener/app/testing-website/3a4d20158ad2c734/49cb56d533c1772b"
                    sh """
                    ./run-stack.sh example-webapp-test ${TEST_ALB_LISTENER_ARN}
                    """
                }
                echo 'Running tests on the integration test environment'
                script {
                    sh """
                       curl -v http://testing-website-1317230480.us-east-1.elb.amazonaws.com | grep '<title>Welcome to example-webapp</title>'
                       if [ \$? -eq 0 ]
                       then
                           echo tests pass
                       else
                           echo tests failed
                           exit 1
                       fi
                    """
                }
            }
        }

 
        stage('Deploy to Production') {
            when {
                branch 'release'
            }
            steps {
                script {
                    PRODUCTION_ALB_LISTENER_ARN="arn:aws:elasticloadbalancing:us-east-1:089778365617:listener/app/production-website/a0459c11ab5707ca/5d21528a13519da6"
                    sh """
                    ./run-stack.sh example-webapp-production ${PRODUCTION_ALB_LISTENER_ARN}
                    """
                }
            }
        }
    }
}