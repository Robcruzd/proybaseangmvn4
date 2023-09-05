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
        CLUSTER_NAME = 'proybasecluster'
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

        // stage('Deliver') {
        //     steps {
        //         sh "echo \$CONTAINER_REGISTRY_PASSWORD | docker login \$CONTAINER_REGISTRY --username \$CONTAINER_REGISTRY_USERNAME --password-stdin"
        //         sh "./mvnw package -Pprod verify jib:build -Djib.to.image=\$CONTAINER_REGISTRY/\$IMAGE_REPOSITORY:\$BUILD_NUMBER"

        //     }
        // }

        // stage('Deploy') {
        //     environment {
        //         AZURE_CLIENT_ID = credentials('AZURE_CLIENT_ID')
        //         AZURE_CLIENT_SECRET = credentials('AZURE_CLIENT_SECRET')
        //         AZURE_TENANT_ID = credentials('AZURE_TENANT_ID')
        //     }
        //     agent any
        //     steps {
        //         sh "az login --service-principal --username \$AZURE_CLIENT_ID --password \$AZURE_CLIENT_SECRET --tenant \$AZURE_TENANT_ID"
        //         sh "az webapp create --resource-group \"\$AZURE_RESOURCE_GROUP\" --plan \"\$AZURE_APP_SERVICE_PLAN\" --name \"\$AZURE_APP_NAME\" --deployment-container-image-name \"\$CONTAINER_REGISTRY/\$IMAGE_REPOSITORY:\$BUILD_NUMBER\""
        //         sh 'az logout'
        //     }
        // }

        stage('Deploy') {
            steps {
                checkout scm
                sh '''
                    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
                    curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
                    chmod +x ./kubectl
                    sudo mv ./kubectl /usr/local/bin/kubectl
                '''
                sh "echo \"\${CONTAINER_REGISTRY_PASSWORD}\" | docker login --username \${CONTAINER_REGISTRY_USERNAME} --password-stdin \${CONTAINER_REGISTRY}"
                sh "az login --service-principal --username \$AZURE_CLIENT_ID --password \$AZURE_CLIENT_SECRET --tenant \$AZURE_TENANT_ID"
                sh "az aks get-credentials --resource-group \"\$AZURE_RESOURCE_GROUP\" --name \"\$CLUSTER_NAME\""
                sh '''
                    kubectl apply -f deployment.yml -f service.yml
                    kubectl set image deployment/robcruzdproybaseangmvn $CONTAINER_REGISTRY_USERNAME=$CONTAINER_REGISTRY/$IMAGE_REPOSITORY:51
                '''
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
