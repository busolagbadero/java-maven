#!/usr/bin/env groovy

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
                    withCredentials([usernamePassword(credentialsId: 'github-cred',passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh 'git config --global user.email "now@gmail.com"'
                        sh 'git config --global user.name "busolami"'
                        sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/busolagbadero/java-maven-version.git"
                        sh 'git add .'
                        sh 'git commit -m "jenkins-git"'
                        sh 'git push'
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
