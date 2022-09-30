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
      image: 'gradle:jdk8',
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
  ],
  volumes: [ 
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'), 
    secretVolume(mountPath: '/.kube/', secretName: 'kube-config'),
  ]
)
{
    node('jenkins-slave-pod') { 
        stage('Git Checkout') {
            container('gradle') {
                git branch: 'main', credentialsId: 'github_cred', url: 'https://github.com/hony7410/cicd_pipeline_test.git'
            }
        }
        stage('Compile') {
            container('gradle') {
                sh "./gradlew compileJava"
            }
        }
        stage('Unit test') {
            container('gradle') {
                sh "./gradlew test"
            }
        }
        stage('Package') {
            container('gradle') {
                sh "./gradlew build"
            }
        }
        stage("Docker build") {
            steps {
                sh "docker build -t test/calculator ."
            }
        }
    }   
}