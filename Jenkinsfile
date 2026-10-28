
pipeline {
    agent any

    environment {
        NAMESPACE = 'default'
        GITHUB_REPO = 'https://github.com/444-debug/r-9.git'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main',
                    url: "${GITHUB_REPO}"
            }
        }

        stage('Refresh Kubernetes Connection') {
            steps {
                bat '''
                    if not exist "C:\\Windows\\System32\\config\\systemprofile\\.kube" mkdir "C:\\Windows\\System32\\config\\systemprofile\\.kube"

                    copy /Y "C:\\Users\\domin\\.kube\\config" "C:\\Windows\\System32\\config\\systemprofile\\.kube\\config"
                    if errorlevel 1 exit /b 1

                    kubectl get nodes
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Create PersistentVolumeClaim') {
            steps {
                bat '''
                    if exist "k8s\\pvc.yaml" (
                        set "PVC_FILE=k8s\\pvc.yaml"
                    ) else if exist "pvc.yaml" (
                        set "PVC_FILE=pvc.yaml"
                    ) else (
                        echo ERROR: pvc.yaml not found in root or k8s folder.
                        dir
                        exit /b 1
                    )

                    kubectl apply -n %NAMESPACE% -f "%PVC_FILE%"
                    if errorlevel 1 exit /b 1

                    kubectl get pvc jenkins-demo-pvc -n %NAMESPACE%
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Create Pod and Mount PVC') {
            steps {
                bat '''
                    if exist "k8s\\pod.yaml" (
                        set "POD_FILE=k8s\\pod.yaml"
                    ) else if exist "pod.yaml" (
                        set "POD_FILE=pod.yaml"
                    ) else (
                        echo ERROR: pod.yaml not found in root or k8s folder.
                        dir
                        exit /b 1
                    )

                    kubectl apply -n %NAMESPACE% -f "%POD_FILE%"
                    if errorlevel 1 exit /b 1

                    kubectl wait -n %NAMESPACE% --for=condition=Ready pod/pvc-demo-pod --timeout=180s
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Verify Volume Mount') {
            steps {
                bat '''
                    kubectl exec -n %NAMESPACE% pvc-demo-pod -- sh -c "echo Created by Jenkins >> /data/jenkins-demo.txt && cat /data/jenkins-demo.txt"
                    if errorlevel 1 exit /b 1

                    kubectl exec -n %NAMESPACE% pvc-demo-pod -- ls -l /data
                    if errorlevel 1 exit /b 1

                    kubectl get pvc,pod -n %NAMESPACE% -o wide
                '''
            }
        }
    }

    post {
        success {
            echo 'PVC created and mounted inside the Pod successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}
