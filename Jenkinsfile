 podTemplate(label: 'maven-persistent',
                    cloud: 'openshift',
                    inheritFrom: 'maven',
                    name: 'maven-persistent',
                    volumes: [persistentVolumeClaim(mountPath: '/home/jenkins/.m2', claimName: 'maven-claim', readOnly: false) ]
              ) {
  node("maven-persistent") {
      
   stage('Checkout Source') {
       checkout scm
    }
    stage('Build App') {
        sh "mvn install"
    }

    stage('Build Image') {
        script {
          openshift.withCluster() {
            openshift.selector("bc", "mapit").startBuild("--from-file=target/mapit-spring.jar", "--wait")
          }
        }
    }
    stage('Promote to DEV') {
        script {
          openshift.withCluster() {
            openshift.tag("mapit:latest", "mapit:dev")
          }
        }
    }

    stage('Promote STAGE') {
        script {
          openshift.withCluster() {
            openshift.tag("mapit:dev", "mapit:stage")
          }
        }
    }
  }
}
