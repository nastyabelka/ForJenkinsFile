pipeline {
    agent {
        label "master"
    }
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "Maven3"
    }
    environment {
    
        
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "10.192.20.238:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "GW-repository-Nexus"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus3"
    }
    stages {
        stage("clone code") {
            steps {
                script {
                    // Let's clone the source
                    checkout([$class: 'GitSCM', 
                    branches: [[name: '*/2.1.x']],
                    userRemoteConfigs: [[url: 'https://github.com/nastyabelka/spring-boot.git']]])
                    sh "ls"
                }
            }
        }
        stage("mvn build") {
            steps {
                script {
                    // If you are using Windows then you should use "bat" step
                    // Since unit testing is out of the scope we skip them
                    sh "ls"
                    sh "pwd"
                    sh "cd spring-boot-samples"
                    sh "ls"
                    sh "mvn -DskipTests clean install -f ./spring-boot-samples/spring-boot-sample-web-ui/pom.xml"
                    sh "ls"
                    sh "find -name *.jar"
                }
            }
        }
        
        stage('Artifact Upload') {
            steps {
                    nexusArtifactUploader artifacts: [
                        [
                            artifactId: 'spring-boot-build', 
                            classifier: '', 
                            file: './spring-boot-samples/spring-boot-sample-web-ui/target/spring-boot-sample-web-ui-2.1.16.BUILD-SNAPSHOT.jar', 
                            type: 'jar'
                        ]
                    ], 
                    credentialsId: 'nexus3', 
                    groupId: 'org.springframework.boot', 
                    nexusUrl: '10.192.20.238:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'GW-repository-Nexus', 
                    version: '4.0.0'
            }    
        }        
                            
        stage("clone dockerfile") {
            steps {
                script {
                    // Let's clone the source
                    git 'https://github.com/nastyabelka/Dockerfile.git';
                }
            }
        }
        
        stage('Build image') {
            steps {
                script {
                    sh '''cd /var/lib/jenkins/workspace/CI-pipeline
                          pwd
                          docker build -t gw:${BUILD_NUMBER} .
                          docker tag gw:${BUILD_NUMBER} 192608593085.dkr.ecr.us-east-1.amazonaws.com/gw:${BUILD_NUMBER}'''
                }
            }    
        } 
        
        stage('Push image') {
            steps {
                script {
                    docker.withRegistry('https://192608593085.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:ecr-user') {
                    sh "docker push 192608593085.dkr.ecr.us-east-1.amazonaws.com/gw:${BUILD_NUMBER}"
                    }
                }
            }    
        }
        
        
        stage('Deploy to ECS') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'Me', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        sh "sed -i 's/NNNNN/${BUILD_NUMBER}/g' task-definition.json"
                        sh '''
                              
                              cat task-definition.json
                              echo 'update task definition...'
                              aws ecs register-task-definition --cli-input-json file:///var/lib/jenkins/workspace/CI-pipeline/task-definition.json --region us-east-1
                              aws ecs update-service --cluster Belka-CI --service Belka-CI-service-last --task-definition Belka-CI-last:1
                        '''
                    
                        
                    }
                }
            }
            
            
        }
        
    }
    post { 
        always { 
            deleteDir()
        }
    }
}
