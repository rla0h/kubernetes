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
                    /*
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
                    */
                    def podInfo = sh(script: 'kubectl get pods -o=jsonpath=\'{range .items[*]}{.metadata.name}{"\\t"}{.status.podIP}{"\\n"}{end}\'', returnStdout: true).trim()
                    podInfo = podInfo.replaceAll("(?m)^\\s*(.*)\\t(.*)\\s*\$", '$1 $2')
                    
                    def fPath = '/home/pin/pod-info.txt'
                    writeFile file: fPath, text: podInfo
                    archiveArtifacts artifacts: fPath, onlyIfSuccessful: false
                }
            }
        }
        //stage('Apply on Pods') {
        //    steps { 
        //        script {
        //            def yamlFilePath = '/home/pin/my-pod.yaml'
        //            sh "kubectl apply -f ${yamlFilePath}"
        //        }
        //    }
        //}

        /*
        stage('Apply Source') {
            steps {
                script {
                    def pub_source = '/home/pin/NWT_TestPublisher.java'
                    def podName = sh(script: "kubectl get pods -o name | grep opendds-pub* | cut -d/ -f 2", returnStdout: true).trim()                   
                    sh "kubectl cp ${pub_source} ${podName}:../NWT/src/NWT_TestPublisher.java"
                    sh "kubectl exec -it ${podName} -- sh -c 'javac -cp classes:/DDS/NWT/lib/*:/DDS/NWT/bin:classes ../NWT/src/NWT_TestPublisher.java'"
                    sh "kubectl exec -it ${podName} -- sh -c 'rm /DDS/NWT/bin/NWT_TestPublisher.class'"
                    sh "kubectl exec -it ${podName} -- sh -c 'mv /DDS/NWT/src/NWT_TestPublisher.class /DDS/NWT/bin/'"
                    //sh "kubectl exec -it ${podName} -- sh -c 'cat ${pub_source} > hi.java'"
                }
            }
        }*/

    }
}