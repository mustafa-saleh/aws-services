library identifier: 'my-shared-library@main', retriever: modernSCM([
    $class: 'GitSCMSource',
    remote: 'https://github.com/mustafa-saleh/demo-module-8-jenkins-shared-library.git',
    credentialsId: 'github-repo'
])

def gv

pipeline {
    agent any

    tools {
        maven 'maven-3.9.16'
    }

    // environment {
    //     IMAGE_NAME = 'mustafa199b/demo:java-maven-1.0'
    // }

    stages {
        stage('increment version') {
            steps {
                script {
                    echo "incrementing the version..."
                    sh 'mvn build-helper:parse-version versions:set \
                    -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                    versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    echo "new version is: ${version}"
                    env.IMAGE_NAME = "mustafa199b/demo:java-maven-$version-$BUILD_NUMBER"
                }
            }
        }

        stage('build jar') {
            steps {
                script {
                    buildJar()
                }
            }
        }

        stage('build image') {
            steps {
                script {
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        }

        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    // def dockerCmd = "docker run -p 8080:8080 -d ${IMAGE_NAME}"
                    // def dockerComposeCmd = 'docker-compose --detach -f docker-compose.yaml up'
                    def ec2Instance = "ec2-user@3.81.125.74"
                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                    
                    sshagent(['ec2-server-key']) {
                        sh "scp server-cmds.sh ${ec2Instance}:/home/ec2-user"
                        sh "scp docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
                }
            }               
        }

        stage('commit version update') {
            steps {
                script {
                    echo "incrementing the version..."
                    withCredentials([gitUsernamePassword(credentialsId: 'github-pass-token', gitToolName: 'Default')]) {
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "Jenkins"'
                        
                        sh 'git add .'
                        sh "git commit -m \"ci: Increment version to ${IMAGE_NAME}\""
                        sh 'git push origin HEAD:main'
                    }
                }
            }
        }
    }
}
