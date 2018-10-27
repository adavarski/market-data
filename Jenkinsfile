def withPod(body) {
  podTemplate(label: 'pod', serviceAccount: 'jenkins', containers: [
      containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
      containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl', command: 'cat', ttyEnabled: true)
    ],
    volumes: [
      hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    ]
 ) { body() }
}

withPod {
  node('pod') {
    def tag = "${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
    def service = "market-data:${tag}"
    def tagToDeploy = "market-data:${env.BUILD_NUMBER}"


    checkout scm

    container('docker') {
      stage('Build') {
        sh("docker build -t ${service} .")
      }
      
      stage('Test') {
         sh("docker run --rm ${service} python setup.py test")
      }
      
      stage('Publish') {
           withDockerRegistry(registry: [credentialsId: 'dockerhub']) {
           sh("docker tag ${service} davarski/${tagToDeploy}")
           sh("docker push davarski/${tagToDeploy}")
      }
      }

 }

    stage('Deploy') {
      sh("sed -i.bak 's#BUILD_TAG#${tagToDeploy}#' ./deploy/staging/*.yml")
   
    container('kubectl') {
        sh("kubectl --namespace=staging apply -f ./deploy/staging")

  }
}
    
 }
}

