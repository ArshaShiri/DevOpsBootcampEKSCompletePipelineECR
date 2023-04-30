pipeline {
    agent any
    tools {
        maven 'maven-3.8.6'
    }
    environment {
        DOCKER_REPO_SERVER='849690659475.dkr.ecr.eu-central-1.amazonaws.com'
        DOCKER_REPO="${DOCKER_REPO_SERVER}/java-maven-app"
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo "incrementing app version..."

                    // This command will update the version in pom.xml
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'

                    // Matcher will contain an array of matched text
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }

        stage('build app') {
            steps {
                script {
                    echo "Building the application..."
                    sh 'mvn clean package'
                    sh 'mvn package'
                }
            }
        }

        stage('build image') {
            steps {
                script {
                    echo "Building the docekr image..."
                    withCredentials([usernamePassword(credentialsId: 'ecr-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t ${IMAGE_REPO}:${DOCKER_REPO} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}"
                        sh "docker push ${IMAGE_REPO}:${DOCKER_REPO}"
                    }
                }
            }
        }

        stage('deploy') {
            environment {
               // The IAM authenticator will need these two environemnt variables to be able to access
               // AWS.
               AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
               AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
               APP_NAME = 'java-maven-app'
            }
            steps {
                script {
                   echo 'deploying docker image...'

                   // Using envsubst, we can substitude the env variables with their actual name.
                   sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -'
                   sh 'envsubst < kubernetes/service.yaml | kubectl apply -f -'
                }
            }
        }
    }
}