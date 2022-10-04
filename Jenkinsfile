podTemplate(label: 'jenkins-slave-pod', 
  containers: [
    containerTemplate(
      name: 'jnlp',
      image: 'jenkins/inbound-agent:4.6-1',
      args: '${computer.jnlpmac} ${computer.name}',
      ttyEnabled: true
    ),
    containerTemplate(
      name: 'gradle',
      image: 'gradle:7.5.1-jdk11',
      command: 'cat',
      ttyEnabled: true,
      runAsUser: '0',
    ),
    containerTemplate(
      name: 'kubectl',
      image: 'bitnami/kubectl',
      command: 'cat',
      ttyEnabled: true,
      runAsUser: '0',
    ),
    containerTemplate(
      name: 'kustomize',
      image: 'k8s.gcr.io/kustomize/kustomize:v3.8.7',
      command: 'cat',
      ttyEnabled: true,
      runAsUser: '0',
    ),
    containerTemplate(
      name: 'docker',
      image: 'docker',
      command: 'cat',
      ttyEnabled: true,
      runAsUser: '0',
    ),
  ],
  volumes: [ 
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'), 
    secretVolume(mountPath: '/.kube/', secretName: 'kube-config'),
    secretVolume(mountPath: '/root/.docker_bk/', secretName: 'regcred'),
  ]
)
{
    node('jenkins-slave-pod') { 
        stage('Git Checkout') {
            container('gradle') {
                git branch: 'main', credentialsId: 'github_cred', url: '${SAMPLE_GIT_URL}'
            }
        }
        stage('Compile') {
            container('gradle') {
                sh "chmod 755 gradlew && ./gradlew compileJava --no-watch-fs"
            }
        }
        stage('Unit test') {
            container('gradle') {
                sh "./gradlew test --no-watch-fs"
            }
        }
        stage('Package') {
            container('gradle') {
                sh "./gradlew build --no-watch-fs"
            }
        }
        stage("Docker build & push") {
            container('docker') {
                sh "mkdir -p /root/.docker/"
                sh "cp /root/.docker_bk/.dockerconfigjson /root/.docker/config.json"
                sh "docker build --tag ${CONTAINER_REGISTRY}/calculator:${env.BUILD_NUMBER} ."
                sh "docker push ${CONTAINER_REGISTRY}/calculator:${env.BUILD_NUMBER}"
            }
        }
        stage("Kustomize New Image & Tag") {
            container('kustomize') {
                sh "cd k8s && /app/kustomize edit set image argoproj/rollouts-demo=${CONTAINER_REGISTRY}/calculator:${env.BUILD_NUMBER}"
            }
        }
        stage("Kubernetes Deploy (canary)") {
            container('kubectl') {
                sh 'kubectl apply -k k8s/'
            }
        }
        stage("Canary Rollout Promote Confirm") {
            input "rollout promote"
        }
        stage("Kubernetes Deploy (Change canary to stable)") {
            container('kubectl') {
                sh "curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64"
                sh "chmod +x ./kubectl-argo-rollouts-linux-amd64"
                sh "mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts"
                sh 'kubectl argo rollouts -n cicd promote rollouts-my-app'
            }
        }
    }   
}