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
                    def pub_podIP = sh(script: 'kubectl get pods -l app=pub -o json | grep -o \'"podIP": "[^"]*\' | awk -F \'"podIP": "\' \'{print $2}\'', returnStdout: true).trim()
                    //def podIPList = podIPs.split("\n")
                    
                    echo "Pub_Pod IPs:"
                    echo pub_podIP

                    def sub_podIP = sh(script: 'kubectl get pods -l app=sub -o json | grep -o \'"podIP": "[^"]*\' | awk -F \'"podIP": "\' \'{print $2}\'', returnStdout: true).trim()
                    
                    echo "Sub_Pod IPs:"
                    echo sub_podIP
                    def sub = sub_podIP

                    echo "value of variable : ${sub}"

                    //for (podIP in podIPList) {
                    //    echo podIP
                    //}
                }
            }
        }

    }
}