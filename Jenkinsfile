def withPod(label, body) {
  podTemplate(label: label, serviceAccount: 'jenkins', containers: [
      containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
      containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.13', command: 'cat', ttyEnabled: true)
    ],
    volumes: [
      hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    ]
 ) { body() }
}

def label = "pod-${UUID.randomUUID().toString()}"

withPod(label) {
  node(label) {
    // def tag = "${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
    def service = "market-data:master.29"
    def tagToDeploy = "morganjbruce/market-data:master.29"

    checkout scm
    //
    // container('docker') {
    //   stage('Build') {
    //     sh("docker build -t ${service} .")
    //   }
    //
    //   stage('Test') {
    //     try {
    //       sh("docker run -v `pwd`:/workspace --rm ${service} python setup.py test")
    //     } finally {
    //       step([$class: 'JUnitResultArchiver', testResults: 'results.xml'])
    //     }
    //   }
    //
    //   stage('Publish') {
    //     withDockerRegistry(registry: [credentialsId: 'dockerhub']) {
    //       sh("docker tag ${service} ${tagToDeploy}")
    //       sh("docker push ${tagToDeploy}")
    //     }
    //   }
    // }

    stage('Deploy') {
      sh("sed -i.bak 's#BUILD_TAG#${tagToDeploy}#' ./deploy/staging/*.yml")

      container('kubectl') {
        sh("kubectl --namespace=staging apply -f deploy/staging")

        try {
          sh("kubectl --namespace=staging --server=https://192.168.99.100:8443 rollout status --request-timeout=5m deployment/market-data")
        } catch(Exception e) {
          sh("kubectl --namespace=staging rollout undo deployment/market-data")
          throw e
        }
      }
    }
  }
}
