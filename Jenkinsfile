podTemplate(containers: [
    containerTemplate(name: 'golang', image: 'uhub.service.ucloud.cn/uk8sdemo/golang:1.16.12', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'kubectl', image: 'uhub.service.ucloud.cn/uk8sdemo/kubectl:latest', command: 'cat', ttyEnabled: true),
  ],
  yaml: """\
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: uhub.service.ucloud.cn/uk8sdemo/executor:debug
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
    - name: kaniko-secret
      secret:
        secretName: regcred
    """.stripIndent()
  ) {
    node(POD_LABEL) {
        stage('Clone') {
            git credentialsId: 'a9245f23-3644-4bc1-8ff3-9edd068958c9', url: 'https://github.com/DennnnyX/kaniko.git'
        }
        stage('Compile') {
            container('golang') {
                    sh """
                    make  
                    """
            }
        }
        stage('Build Image')
            container('kaniko') {
                sh """
                /kaniko/executor -c `pwd`/ -f `pwd`/image/Dockerfile -d dennnys/pipeline:v2
                """
            }
       stage('Deploy') {
         withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]){
           container('kubectl') {
           sh """
              mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config
              kubectl apply -f deploy/k8s.yaml 
            """
            
           }
         }
       }
    }
}
