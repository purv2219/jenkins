#!/usr/bin/env groovy

pipeline {
    agent any
    environment {
        NEW_VERSION = '1.3'
    }

    parameters {
        choice(name: 'VERSION', choices:['1','2', '3'], description: '')
        booleanParam(name: 'executeTest', defaultValue : true, description: '')
    }
    
    tools{
        maven 'maven-3.9.0'
    }

    stages {
        stage('init'){
            steps{
                script{
                    gv = load "script.groovy"
                }
            }
        }
        stage('config'){
            steps{
                script{
                    gv.config()
                }
            }
        }
        stage('build') {
            
            steps {
                script{
                    echo 'Building The Application'
                    echo "Software Version is ${NEW_VERSION}"
                    sh 'mvn build-helper:parse-version versions:set -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.nextMinorVersion}.\\\${parsedVersion.incrementalVersion}\\\${parsedVersion.qualifier?}' 
                    sh 'mvn clean package'
                    def version = (readFile('pom.xml') =~ '<version>(.+)</version>')[0][2]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                    sh "docker build -t purv22/spring-boot:${IMAGE_NAME} ."
                        
                    }
            }
        }
      stage('test') {
          when{  
             expression{
                 params.executeTest
             }
          }
            steps {
                script{echo 'Testing the application'
                sh 'mvn test'}
            }
        }
      stage('deploy') {
        input{
            message "Select the environment to deploy"
            ok "done"
            parameters{
                choice(name: 'Type', choices:['Dev','Test','Deploy'], description: '')
            }

        }
            steps {
                script{echo 'Deploying The Application'
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                    sh "echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin"
                    sh "docker push purv22/spring-boot:${IMAGE_NAME}"
                }}
                
             }
        }
//         stage('commit version update'){
//             steps{
//                 script{
//                     withCredentials([usernamePassword(credentialsId: 'git-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
//                         sh 'git config --global user.email "jenkins@example.com"'
//                         sh 'git config --global user.name "jenkins"'

//                         sh 'git status'
//                         sh 'git branch'
//                         sh 'git config --list'

//                         sh "git remote set-url origin https://${USERNAME}:${PASSWORD}@github.com/learnwithparth/springboot-jenkins.git"
//                         sh 'git add .'
//                         sh 'git commit -m "version change"'
//                         sh 'git push origin HEAD:jenkins-jobs'
//                     }
//                 }
//             }
//         }
    }
    post{
        always{
            echo 'Executing always...'
        }
        success{
            echo 'Executing success'
        }
        failure{
            echo 'Executing failure'
        }
    }
}
