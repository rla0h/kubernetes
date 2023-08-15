pipeline {
    agent any

    environment {
        KUBECONFIG = '/home/pin/kube_adm.conf'
        pub_pod = '/home/pin/pub-statefulset.yaml'
        sub_pod = '/home/pin/sub-statefulset.yaml'
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
                    def pub = pub_podIP
                    echo "value of variable : ${pub}, ${sub}"

                    //for (podIP in podIPList) {
                    //    echo podIP
                    //}
                }
            }
        }
        stage('Apply on Pods') {
            steps { 
                script {
                    def yamlFilePath = '/home/pin/my-pod.yaml'
                    sh "kubectl apply -f ${yamlFilePath}"
                }
            }
        }
        stage('Apply Source') {
            steps {
                script {
                    def pub_source = '/home/pin/NWT_TestPublisher.java'
                    def podName = sh(script: "kubectl get pods -o name | grep my* | cut -d/ -f 2", returnStdout: true).trim()
                    echo "${podName}"
                    sh "kubectl exec -it ${podName} -- sh -c 'echo \"hello\" > ./hello.txt'"
                }
            }
        }

    }
}