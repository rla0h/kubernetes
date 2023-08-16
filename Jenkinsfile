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

        stage('Get Pod IPs and Apply to /etc/hosts (on Pub & Sub)') {
            steps {
                script {
                    def podInfo = sh(script: 'kubectl get pods -o=jsonpath=\'{range .items[*]}{.status.podIP}{"\\t"}{.metadata.name}{"\\n"}{end}\'', returnStdout: true).trim()
                    podInfo = podInfo.replaceAll("(?m)^\\s*(.*)\\t(.*)\\s*\$", '$1 $2')
                    def pubpodName = sh(script: "kubectl get pods -o name | grep opendds-pub* | cut -d/ -f 2", returnStdout: true).trim()
                    def subpodName = sh(script: "kubectl get pods -o name | grep opendds-sub* | cut -d/ -f 2", returnStdout: true).trim()
                    
                    def fPath = 'pod-info.txt'
                    writeFile file: fPath, text: podInfo
                    
                    sh "kubectl cp ${fPath} ${pubpodName}:./pod-info.txt"
                    sh "kubectl cp ${fPath} ${subpodName}:./pod-info.txt"
                    sh "kubectl exec -it ${pubpodName} -- sh -c 'cat pod-info.txt >> /etc/hosts'"
                    sh "kubectl exec -it ${subpodName} -- sh -c 'cat pod-info.txt >> /etc/hosts'"
                }
            }
        }
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
                    /*
                    def pubcombinedInfo = "${pubserviceIP}\t${pubpodNames}\n"
                    def subcombinedInfo = "${subserviceIP}\t${subpodNames}\n"

                    def pservicepath = 'pub-service-ip.txt'
                    def sservicepath = 'sub-service-ip.txt'
                    writeFile file: ${pservicepath}, text: pubcombinedInfo
                    writeFile file: ${sservicepath}, text: subcombinedInfo

                    sh "kubectl cp ${pservicepath} ${repopodName}:./pub-service.txt'"
                    sh "kubectl cp ${sservicepath} ${repopodName}:./sub-service.txt'"

                    sh "kubectl exec -it ${repopodName} -- sh -c 'cat pub-service.txt >> /etc/hosts'"
                    sh "kubectl exec -it ${repopodName} -- sh -c 'cat sub-service.txt >> /etc/hosts'"*/
                    sh "kubectl exec -it ${repopodName} -- sh -c '/DDS/OpenDDS-DDS-3.23.1/bin/DCPSInfoRepo -ORBListenEndpoints iiop://${reposerviceIP}:1212'"
                }
            }
        }

        
        stage('Apply Source Pub') {
            steps {
                script {
                    def pub_source = '/home/pin/NWT_TestPublisher.java'
                    def podName = sh(script: "kubectl get pods -o name | grep opendds-pub* | cut -d/ -f 2", returnStdout: true).trim()                   
                    sh "kubectl cp ${pub_source} ${podName}:../NWT/src/NWT_TestPublisher.java"
                    sh "kubectl exec -it ${podName} -- sh -c 'javac -cp classes:/DDS/NWT/lib/*:/DDS/NWT/bin:classes ../NWT/src/NWT_TestPublisher.java'"
                    sh "kubectl exec -it ${podName} -- sh -c 'rm /DDS/NWT/bin/NWT_TestPublisher.class'"
                    sh "kubectl exec -it ${podName} -- sh -c 'mv /DDS/NWT/src/NWT_TestPublisher.class /DDS/NWT/bin/'"

                    sh "kubectl exec -it ${podName} -- sh -c '/DDS/NWT/bin/java -ea -cp classes:/DDS/NWT/lib/*:/DDS/NWT/bin:classes -Djava.library.path=$DDS_ROOT/lib NWT_TestPublisher -DCPSConfigFile tcp.ini -DCPSTransportDebugLevel 0 -w'"                    
                }
            }
        }

        stage('Apply Source Sub') {
            steps {
                script {
                    def pub_source = '/home/pin/NWT_TestSubscriber.java'
                    def podName = sh(script: "kubectl get pods -o name | grep opendds-sub* | cut -d/ -f 2", returnStdout: true).trim()                   
                    sh "kubectl cp ${pub_source} ${podName}:../NWT/src/NWT_TestSubscriber.java"
                    sh "kubectl exec -it ${podName} -- sh -c 'javac -cp classes:/DDS/NWT/lib/*:/DDS/NWT/bin:classes ../NWT/src/NWT_TestSubscriber.java'"
                    sh "kubectl exec -it ${podName} -- sh -c 'rm /DDS/NWT/bin/NWT_TestSubscriber.class'"
                    sh "kubectl exec -it ${podName} -- sh -c 'mv /DDS/NWT/src/NWT_TestSubscriber.class /DDS/NWT/bin/'"
                    sh "kubectl exec -it ${podName} -- sh -c '/DDS/NWT/bin/java -ea -cp classes:/DDS/NWT/lib/*:/DDS/NWT/bin:classes -Djava.library.path=$DDS_ROOT/lib NWT_TestSubscriber -DCPSConfigFile tcp.ini -DCPSTransportDebugLevel 0 -r'"
                }
            }
        }

    }
}