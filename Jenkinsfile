pipeline {
    agent any

    environment {
        KUBECONFIG = '/home/pin/kube_adm.conf'
    }
    stages {
        stage('Check the Node') {
            steps {
                script {
                    def kubectlOutput = sh(script: "kubectl --kubeconfig=${env.KUBECONFIG} get pods", returnStdout: true).trim()
                    echo "kubectl get pods output:"
                    echo kubectlOutput
                }
            }
        }

        stage('Get Pod IPs') {
            steps {
                script {
                    def podIPs = sh(script: 'kubectl get pods -o json | grep -o \'"podIP": "[^"]*\' | awk -F \'"podIP": "\' \'{print $2}\'', returnStdout: true).trim()
                    def podIPList = podIPs.split("\n")
                    
                    echo "Pod IPs:"
                    for (podIP in podIPList) {
                        echo podIP
                    }
                }
            }
        }

    }
}