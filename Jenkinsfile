#!/usr/bin/env groovy

pipeline {
    agent any

    stages {
        stage('checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                checkout scm
                sh "java -version"
                sh "chmod +x mvnw"
                sh "./mvnw -ntp clean -P-webapp"
                sh "./mvnw -ntp checkstyle:check"
                sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:install-node-and-npm@install-node-and-npm"
                sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm"
                sh "./mvnw -ntp verify -P-webapp"
                sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm -Dfrontend.npm.arguments='run test'"
            }
        }

        stage('Deliver') {
            steps {
                script {
                    sh "chmod +x mvnw"
                    sh "./mvnw -ntp verify -P-webapp -Pprod -DskipTests"
                    stash name: 'jar', includes: '**/target/*.jar'
                }
            }
        }

        stage('Deploy') {
            environment {
                AZURE_CLIENT_ID = credentials('AZURE_CLIENT_ID')
                AZURE_CLIENT_SECRET = credentials('AZURE_CLIENT_SECRET')
                AZURE_TENANT_ID = credentials('AZURE_TENANT_ID')
            }
            agent any
            steps {
                unstash 'jar'
                script {
                    withCredentials([string(credentialsId: 'AZURE_CLIENT_ID', variable: 'AZURE_CLIENT_ID'),
                                     string(credentialsId: 'AZURE_CLIENT_SECRET', variable: 'AZURE_CLIENT_SECRET'),
                                     string(credentialsId: 'AZURE_TENANT_ID', variable: 'AZURE_TENANT_ID')]) {
                        sh '''
                            find . -name proy*.jar
                            az login --service-principal --username $AZURE_CLIENT_ID --password $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
                            az webapp deploy --resource-group proybase --name proybaseappserv --src-path $(find . -name "proy*.jar") --type jar --verbose
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
