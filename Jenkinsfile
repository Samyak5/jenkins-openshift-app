pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: dc-pipeline
spec:
  volumes:
    - name: dind-storage
      emptyDir: {}
  securityContext:
    runAsNonRoot: false
  containers:
  - name: jnlp
    image: 'jenkins/inbound-agent:3142.vcfca_0cd92128-1'
    imagePullPolicy: Always
    args: ['\$(JENKINS_SECRET)', '\$(JENKINS_NAME)']
    securityContext:
      privileged: true 
      allowPrivilegeEscalation: true     
  - name: docker
    image: docker:24.0-dind
    securityContext:
       privileged: true 
    volumeMounts:
        - name: dind-storage
          mountPath: /var/lib/docker     
    env:
      - name: POD_IP
        valueFrom:
          fieldRef:
            fieldPath: status.podIP
      - name: DOCKER_HOST
        value: tcp://localhost:2375
  - name: openshift-cli
    image: us.icr.io/100mc-assets/openshift/origin-cli:v3.11.0
    command:
    - cat
    tty: true
    securityContext:
        privileged: true 
  serviceAccountName: pipline
  serviceAccount: pipline
  imagePullSecrets:
   - name: regsecret
   - name: all-icr-io        
 """  
        }
    }
    stages {
        stage('Check Docker') {
            steps {
                container('docker') {
                    sh 'docker --version'
                }
            }
        }

        stage('Git Clone') {
            steps {
                container('docker') {
                    git branch: 'main', credentialsId: 'ram-git', url: 'https://github.ibm.com/Nalla-Ramnath/jenkins-test.git'
                    sh 'ls -l'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                container('openshift-cli') {
                    script {
                        sh 'oc get pods'
                        sh 'oc run test-pod-httpd-4 --image=image-registry.openshift-image-registry.svc:5000/openshift/httpd:latest'
                        sh 'oc get pods'
                    }
                }
            }
        }
    }
}
