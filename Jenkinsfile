pipeline {
    agent any
    tools {
        maven 'maven-b'
    }
    stages {
        stage("increment Version") {
            steps {
                script {
                    echo 'incrementing version'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }

        stage("Build App") {
            steps {
                script {
                    echo 'building the app'
                    sh 'mvn clean package'
                }
            }
        }
        stage('Build Image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t gbaderobusola/busola:${IMAGE_NAME} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push gbaderobusola/busola:${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('Push to Git') {
            steps {
                script {
                    echo "Pushing to Git..."
                    withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-ssh', keyFileVariable: 'SSH_KEY_CREDENTIAL')]) {
                        sh 'git remote set-url origin git@github.com:busolagbadero/java-maven-version.git'
                        sh 'git add .'
                        sh 'git commit -m "JENKINSCID to GITD"'
                        sh 'git push origin HEAD:master'
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    echo "Deploying the application..."
                }
            }
        }
    }
}
