pipeline {
    agent any
    environment {
        admKUBECONFIG = '/home/pin/kube_adm.conf'
        pub_pod = '/home/pin/pub-statefulset.yaml'
        sub_pod = '/home/pin/sub-statefulset.yaml'
    }
    stages {
        stage('Check the Node') {
            steps {
                script {
                    def kubectlOutput = sh(script: "kubectl --kubeconfig=${env.admKUBECONFIG} get pods", returnStdout: true).trim()
            
                    def topNodesOutput = sh(script: "kubectl --kubeconfig=${env.admKUBECONFIG} top nodes", returnStdout: true).trim()
                    
                    echo topNodesOutput                
                    echo kubectlOutput
                }
            }
        }
        
        stage('Get Pod IPs and Apply to /etc/hosts (on Pub & Sub)') {
            steps {
                script {
                    def podInfo = sh(script: """
                        kubectl get --kubeconfig=${env.admKUBECONFIG} pods -o=jsonpath=\'{range .items[*]}{.status.podIP}{"\\t"}{.metadata.name}{"\\n"}{end}\'""", returnStdout: true).trim()
                    podInfo = podInfo.replaceAll("(?m)^\\s*(.*)\\t(.*)\\s*\$", '$1 $2')
                    def pubpodName = sh(script: "kubectl --kubeconfig=${env.admKUBECONFIG} get pods -o name | grep opendds-pub* | cut -d/ -f 2", returnStdout: true).trim()
                    def subpodName = sh(script: "kubectl --kubeconfig=${env.admKUBECONFIG} get pods -o name | grep opendds-sub* | cut -d/ -f 2", returnStdout: true).trim()
                    
                    def fPath = 'pod-info.txt'
                    writeFile file: fPath, text: podInfo+"\n"
                    
                    sh "kubectl --kubeconfig=${env.admKUBECONFIG} cp ${fPath} ${pubpodName}:./pod-info.txt"
                    sh "kubectl --kubeconfig=${env.admKUBECONFIG} cp ${fPath} ${subpodName}:./pod-info.txt"
                    sh "kubectl --kubeconfig=${env.admKUBECONFIG} exec -it ${pubpodName} -- sh -c 'cat pod-info.txt >> /etc/hosts'"
                    sh "kubectl --kubeconfig=${env.admKUBECONFIG} exec -it ${subpodName} -- sh -c 'cat pod-info.txt >> /etc/hosts'"
                }
            }
        }
        /*
        stage('Get Service Pod IP and Apply to /etc/hosts (on Repository)') {
            steps {
                script {
                    def pubpodName = sh(script: "kubectl get pods -o name | grep opendds-pub* | cut -d/ -f 2", returnStdout: true).trim()
                    def subpodName = sh(script: "kubectl get pods -o name | grep opendds-sub* | cut -d/ -f 2", returnStdout: true).trim()
                    def repopodName = sh(script: "kubectl get pods -o name | grep reposvc* | cut -d/ -f 2", returnStdout: true).trim()
                    def pubserviceName = 'pub-service'  // Replace with your actual Service name
                    def subserviceName = 'sub-service'
                    def reposerviceName = 'repo-service'
                    

                    def pubserviceIP = sh(script: "kubectl get service ${pubserviceName} --template='{{.spec.clusterIP}}'", returnStdout: true).trim()
                    def subserviceIP = sh(script: "kubectl get service ${subserviceName} --template='{{.spec.clusterIP}}'", returnStdout: true).trim()
                    def reposerviceIP = sh(script: "kubectl get service ${reposerviceName} --template='{{.spec.clusterIP}}'", returnStdout: true).trim()
                    
                    //def pubcombinedInfo = "${pubserviceIP}\t${pubpodNames}\n"
                    //def subcombinedInfo = "${subserviceIP}\t${subpodNames}\n"

                    //def pservicepath = 'pub-service-ip.txt'
                    //def sservicepath = 'sub-service-ip.txt'
                    //writeFile file: ${pservicepath}, text: pubcombinedInfo
                    //writeFile file: ${sservicepath}, text: subcombinedInfo

                    //sh "kubectl cp ${pservicepath} ${repopodName}:./pub-service.txt'"
                    //sh "kubectl cp ${sservicepath} ${repopodName}:./sub-service.txt'"

                    sh "kubectl exec -it ${repopodName} -- sh -c 'cat pub-service.txt >> /etc/hosts'"
                    sh "kubectl exec -it ${repopodName} -- sh -c 'cat sub-service.txt >> /etc/hosts'"
                    sh "kubectl exec -it ${repopodName} -- sh -c '/DDS/OpenDDS-DDS-3.23.1/bin/./DCPSInfoRepo -o /DDS/OpenDDS-DDS-3.23.1/lib/* -ORBListenEndpoints iiop://${reposerviceIP}:1212'"
                }
            }
        }*/

        
        stage('Apply Source Pub') {
            steps {
                script {
                    def pub_source = '/home/pin/NWT_TestPublisher.java'
                    def pubpodName = sh(script: "kubectl --kubeconfig=${env.admKUBECONFIG} get pods -o name | grep opendds-pub* | cut -d/ -f 2", returnStdout: true).trim()                   
                    sh "kubectl --kubeconfig=${env.admKUBECONFIG} cp ${pub_source} ${pubpodName}:../NWT/src/NWT_TestPublisher.java"
                    sh "kubectl --kubeconfig=${env.admKUBECONFIG} exec -it ${pubpodName} -- sh -c 'javac -cp classes:/DDS/NWT/lib/*:/DDS/NWT/bin:classes ../NWT/src/NWT_TestPublisher.java'"
                    sh "kubectl --kubeconfig=${env.admKUBECONFIG} exec -it ${pubpodName} -- sh -c 'rm /DDS/NWT/bin/NWT_TestPublisher.class'"
                    sh "kubectl --kubeconfig=${env.admKUBECONFIG} exec -it ${pubpodName} -- sh -c 'mv /DDS/NWT/src/NWT_TestPublisher.class /DDS/NWT/bin/'"

                    //sh "kubectl exec -it ${pubpodName} -- sh -c 'cd ../NWT/bin && java -ea -cp classes:/DDS/NWT/lib/*:/DDS/NWT/bin:class -Djava.library.path=/DDS/OpenDDS-DDS-3.23.1/lib/ NWT_TestPublisher -DCPSConfigFile tcp.ini -DCPSTransportDebugLevel 0 -w'"                    
                }
            }
        }

        stage('Apply Source Sub') {
            steps {
                script {
                    def sub_source = '/home/pin/NWT_DataReaderListenerImpl.java'
                    def subpodName = sh(script: "kubectl --kubeconfig=${env.admKUBECONFIG} get pods -o name | grep opendds-sub* | cut -d/ -f 2", returnStdout: true).trim()                   
                    sh "kubectl --kubeconfig=${env.admKUBECONFIG} cp ${sub_source} ${subpodName}:../NWT/src/NWT_DataReaderListenerImpl.java"
                    sh "kubectl --kubeconfig=${env.admKUBECONFIG} exec -it ${subpodName} -- sh -c 'javac -cp classes:/DDS/NWT/lib/*:/DDS/NWT/bin:classes ../NWT/src/NWT_DataReaderListenerImpl.java'"
                    sh "kubectl --kubeconfig=${env.admKUBECONFIG} exec -it ${subpodName} -- sh -c 'rm /DDS/NWT/bin/NWT_DataReaderListenerImpl.class'"
                    sh "kubectl --kubeconfig=${env.admKUBECONFIG} exec -it ${subpodName} -- sh -c 'mv /DDS/NWT/src/NWT_DataReaderListenerImpl.class /DDS/NWT/bin/.'"
                    //sh "kubectl exec -it ${subpodName} -- sh -c 'cd ../NWT/bin && java -ea -cp classes:/DDS/NWT/lib/*:/DDS/NWT/bin:classes -Djava.library.path=/DDS/OpenDDS-DDS-3.23.1/lib/ NWT_TestSubscriber -DCPSConfigFile tcp.ini -DCPSTransportDebugLevel 0 -r'"
                }
            }
        }

    }
}