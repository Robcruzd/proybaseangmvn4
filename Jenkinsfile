#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        CONTAINER_REGISTRY_PASSWORD = 'n4yPSHC3s9lxYOfGXB3joVxVOssE7+vfrXcLGuGFr5+ACRDQeGic'
        CONTAINER_REGISTRY = 'proybaseazcr.azurecr.io'
        CONTAINER_REGISTRY_USERNAME = 'proybaseazcr'
        IMAGE_REPOSITORY = 'proybase_image'
        AZURE_CLIENT_ID = credentials('AZURE_CLIENT_ID')
        AZURE_CLIENT_SECRET = credentials('AZURE_CLIENT_SECRET')
        AZURE_TENANT_ID = credentials('AZURE_TENANT_ID')
        AZURE_RESOURCE_GROUP = 'proybase'
        AZURE_APP_SERVICE_PLAN = 'proybaseappservplan'
        AZURE_APP_NAME = 'proybaseappservcon'
    }

    stages {
        stage('checkout') {
            steps {
                checkout scm
            }
        }

        // stage('Build and Test') {
        //     steps {
        //         checkout scm
        //         sh "java -version"
        //         sh "chmod +x mvnw"
        //         sh "./mvnw -ntp clean -P-webapp"
        //         sh "./mvnw -ntp checkstyle:check"
        //         sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:install-node-and-npm@install-node-and-npm"
        //         sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm"
        //         sh "./mvnw -ntp verify -P-webapp"
        //         sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm -Dfrontend.npm.arguments='run test'"
        //     }
        // }

        stage('Deliver') {
            steps {
                withCredentials([string(credentialsId: 'CONTAINER_REGISTRY_PASSWORD', variable: 'CONTAINER_REGISTRY_PASSWORD')]) {
                    sh "echo \$CONTAINER_REGISTRY_PASSWORD | docker login \$CONTAINER_REGISTRY --username \$CONTAINER_REGISTRY_USERNAME --password-stdin"
                }
                def timestamp = new Date().format("yyyyMMddHHmmss")
                sh "./mvnw package -Pprod verify jib:build -Djib.to.image=\$CONTAINER_REGISTRY/\$IMAGE_REPOSITORY:\$timestamp"

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
                withCredentials([
                    string(credentialsId: 'AZURE_CLIENT_ID', variable: 'AZURE_CLIENT_ID'),
                    string(credentialsId: 'AZURE_CLIENT_SECRET', variable: 'AZURE_CLIENT_SECRET'),
                    string(credentialsId: 'AZURE_TENANT_ID', variable: 'AZURE_TENANT_ID')
                ]) {
                    sh "az login --service-principal --username \$AZURE_CLIENT_ID --password \$AZURE_CLIENT_SECRET --tenant \$AZURE_TENANT_ID"
                }
                def timestamp = new Date().format("yyyyMMddHHmmss")
                sh "az webapp create --resource-group \"\$AZURE_RESOURCE_GROUP\" --plan \"\$AZURE_APP_SERVICE_PLAN\" --name \"\$AZURE_APP_NAME\" --deployment-container-image-name \"\$CONTAINER_REGISTRY/\$IMAGE_REPOSITORY:\$timestamp\""
                sh 'az logout'
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
