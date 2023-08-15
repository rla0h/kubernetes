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
                    def yamlContent = """
                    apiVersion: apps/v1
kind: StatefulSet
metadata:
    name: opendds-sub
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sub
  template:
    metadata:
      labels:
        app: sub
    spec:
      nodeName: worker2
      containers:
      - name: sub
        image: happykimyh/opendds:v1.3
        imagePullPolicy: Always
        securityContext:
          runAsUser: 0
        command: ['sh', '-c', "while :; do echo '.'; sleep 5 ; done"]
        ports:
          - containerPort: 1223
---

apiVersion: v1
kind: Service
metadata:
   name: sub-service
spec:
   selector:
     app: pub
   ports:
    - protocol: TCP
      port: 1223
      """
                    writeFile file: 'my-pod.yaml', text: yamlContent
                    echo "Workspace directory: ${env.WORKSPACE}"
                }
            }
        }

    }
}