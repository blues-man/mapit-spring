 podTemplate(label: 'maven-persistent',
                    cloud: 'openshift',
                    inheritFrom: 'maven',
                    name: 'maven-persistent',
                    volumes: [persistentVolumeClaim(mountPath: '/home/jenkins/.m2', claimName: 'maven-claim', readOnly: false) ]
              ) {
  node("maven-persistent") {
   
   stages {
   
   stage('Checkout Source') {
        git url: "https://github.com/blues-man/mapit-spring.git"
    }
    stage('Build App') {
        sh "mvn install"
    }
    stage('Create Image Builder') {
      when {
        expression {
          openshift.withCluster() {
            return !openshift.selector("bc", "mapit").exists();
          }
        }
      }
        script {
          openshift.withCluster() {
            openshift.newBuild("--name=mapit", "--image-stream=redhat-openjdk18-openshift:1.1", "--binary")
          }
        }
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
    stage('Create DEV') {
      when {
        expression {
          openshift.withCluster() {
            return !openshift.selector('dc', 'mapit-dev').exists()
          }
        }
      }
        script {
          openshift.withCluster() {
            openshift.newApp("mapit:latest", "--name=mapit-dev").narrow('svc').expose()
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
    stage('Create STAGE') {
      when {
        expression {
          openshift.withCluster() {
            return !openshift.selector('dc', 'mapit-stage').exists()
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.newApp("mapit:stage", "--name=mapit-stage").narrow('svc').expose()
          }
        }
      }
    }
   }
  }
}
