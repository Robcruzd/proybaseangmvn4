#!/usr/bin/env groovy

pipeline {
    agent any

    stages {
        // stage('checkout') {
        //     steps {
        //         checkout scm
        //     }
        // }

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
        //         // sh 'npm install'
        //         // sh 'chmod +x mvnw'
        //         // sh 'npm run ci:backend:test'
        //         // sh 'npm run ci:frontend:test'
        //     }
        //     // post {
        //     //     always {
        //     //         saveCache(key: 'v1-dependencies-${CHECKSUM}', paths: ['node', 'node_modules', '.m2'])
        //     //     }
        //     // }
        // }

        stage('Deliver') {
            // agent {
            //     docker {
            //         image 'ubuntu:20.04'
            //     }
            // }
            steps {
                sh 'chmod +x mvnw'
                sh "./mvnw -ntp verify -P-webapp -Pprod -DskipTests"
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
            // post {
            //     always {
            //         archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            //     }
            // }
        }

        stage('Deploy') {
            environment {
                AZURE_CLIENT_ID = credentials('AZURE_CLIENT_ID')
                AZURE_CLIENT_SECRET = credentials('AZURE_CLIENT_SECRET')
                AZURE_TENANT_ID = credentials('AZURE_TENANT_ID')
            }
            agent any
            steps {
                // attachWorkspace()
                script {
                    withCredentials([string(credentialsId: 'AZURE_CLIENT_ID', variable: 'AZURE_CLIENT_ID'),
                                     string(credentialsId: 'AZURE_CLIENT_SECRET', variable: 'AZURE_CLIENT_SECRET'),
                                     string(credentialsId: 'AZURE_TENANT_ID', variable: 'AZURE_TENANT_ID')]) {
                        sh '''
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

// node {

//     stage('checkout') {
//         checkout scm
//     }

//     stage('check java') {
//         sh "java -version"
//     }

//     stage('clean') {
//         sh "chmod +x mvnw"
//         sh "./mvnw -ntp clean -P-webapp"
//     }
//     stage('nohttp') {
//         sh "./mvnw -ntp checkstyle:check"
//     }

//     stage('install tools') {
//         sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:install-node-and-npm@install-node-and-npm"
//     }

//     stage('npm install') {
//         sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm"
//     }
//     stage('backend tests') {
//         try {
//             sh "./mvnw -ntp verify -P-webapp"
//         } catch(err) {
//             throw err
//         } finally {
//             junit '**/target/surefire-reports/TEST-*.xml,**/target/failsafe-reports/TEST-*.xml'
//         }
//     }

//     stage('frontend tests') {
//         try {
//             sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm -Dfrontend.npm.arguments='run test'"
//         } catch(err) {
//             throw err
//         } finally {
//             junit '**/target/test-results/TESTS-results-jest.xml'
//         }
//     }

//     stage('packaging') {
//         sh "./mvnw -ntp verify -P-webapp -Pprod -DskipTests"
//         archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
//     }
// }
