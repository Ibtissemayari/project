pipeline {
    agent any
    
    environment {
        SERVICE_NAME = "project"
        ORGANIZATION_NAME = "ebtissemayari"
        DOCKERHUB_USERNAME = "ebtissemayari"
        REPOSITORY_TAG = "${DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
    }
   
    stages {
            
            
        stage ('Build and Push Image') {
            steps {
                 withDockerRegistry([credentialsId: 'docker-hub', url: ""]) {
                   sh 'docker build -t ${REPOSITORY_TAG} .'
                   sh 'docker push ${REPOSITORY_TAG}'          
            }
          }
       }
        stage("Install kubectl"){
            steps {
                sh """
                    curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
                    chmod +x ./kubectl
                    ./kubectl version --client
                """
            }
        }
        
        stage ('Deploy to Cluster') {
            steps {
                //withAWS(role: "Jenkins", roleAccount: '164135465533') {
                sh "aws eks update-kubeconfig --region eu-central-1 --name eks"
               // sh "aws eks update-kubeconfig --region eu-central-1 --name eks"
                sh " envsubst < ${WORKSPACE}/deploy.yaml | ./kubectl apply -f - "
                sh " envsubst < ${WORKSPACE}/postgres-configmap.yaml | ./kubectl apply -f - "
                sh " envsubst < ${WORKSPACE}/storageclass.yaml | ./kubectl apply -f - "
                sh " envsubst < ${WORKSPACE}/postgres-pv.yaml | ./kubectl apply -f - "
                sh " envsubst < ${WORKSPACE}/claim.yaml | ./kubectl apply -f - "
                sh " envsubst < ${WORKSPACE}/postgres-deploy.yaml | ./kubectl apply -f - "
                sh " envsubst < ${WORKSPACE}/postgres-service.yaml | ./kubectl apply -f - "
                sh " envsubst < ${WORKSPACE}/odoo-config.yaml | ./kubectl apply -f - "
                sh " envsubst < ${WORKSPACE}/odoo-deploy.yaml | ./kubectl apply -f - "
                sh " envsubst < ${WORKSPACE}/odoo-service.yaml | ./kubectl apply -f - "
                //}
            }
        }
    }
}