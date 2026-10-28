pipeline {
    agent any

    environment {
        NAMESPACE = 'default'
    }

    stages {
        stage('Check tools and cluster access') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        set -eu
                        kubectl version --client
                        kubectl --namespace="$NAMESPACE" get nodes
                    '''
                }
            }
        }

        stage('Create PersistentVolumeClaim') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        set -eu
                        kubectl apply -n "$NAMESPACE" -f k8s/pvc.yaml
                        kubectl wait -n "$NAMESPACE" --for=jsonpath='{.status.phase}'=Bound pvc/jenkins-demo-pvc --timeout=180s
                    '''
                }
            }
        }

        stage('Create Pod and mount the claim') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        set -eu
                        kubectl apply -n "$NAMESPACE" -f k8s/pod.yaml
                        kubectl wait -n "$NAMESPACE" --for=condition=Ready pod/pvc-demo-pod --timeout=180s
                    '''
                }
            }
        }

        stage('Verify mounted storage') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        set -eu
                        kubectl exec -n "$NAMESPACE" pvc-demo-pod -- sh -c \
                          'echo "Created by Jenkins on $(date)" >> /data/jenkins-demo.txt && ls -l /data && cat /data/jenkins-demo.txt'
                        echo "PVC and Pod details:"
                        kubectl get pvc,pod -n "$NAMESPACE" -o wide
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Success: the PVC was created and mounted at /data inside pvc-demo-pod.'
        }
        failure {
            echo 'Pipeline failed. Check Kubernetes access, StorageClass availability, and pod events.'
        }
    }
}
