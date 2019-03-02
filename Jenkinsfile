pipeline {
  agent any
  environment {
    PACKAGE_REPO_DIR='/home/cloud/package'
    REMOTE_HOST_IP='192.168.1.77'
    REMOTE_HOST_USER='root'
    REMOTE_HOST_PWD='KINGking_123'
    JENKINS_BIND_IP='192.168.1.77'
    JENKINS_PORT='8081'
  }

  stages {
    stage('build') {
      parallel {
        stage('install docker') {
          steps {
            script {
              def host = [:]
              host.name = 'docker'
              host.host = env.REMOTE_HOST_IP
              host.user = env.REMOTE_HOST_USER
              host.password = env.REMOTE_HOST_PWD
              host.allowAnyHosts = 'true'

              sshCommand remote:host, command:"rm -rf ~/docker-install"
              sshPut remote:host, from:"./install/docker-install", into:"."
              sshCommand remote:host, command:"cd ~/docker-install;sh install.sh --install"
            }
          }
        }

        stage('install jenkins') {
          steps {
            sh '''cd ./install/maven-install; \\
                  echo "PACKAGE_REPO_DIR=${PACKAGE_REPO_DIR}" >> config.properties; \\
                  sh install.sh --package'''
            sh '''cd ./install/jenkins-install; \\
                  echo "PACKAGE_REPO_DIR=${PACKAGE_REPO_DIR}" >> config.properties; \\
                  sh install.sh --package'''

            script {
              def host = [:]
              host.name = 'jenkins'
              host.host = env.REMOTE_HOST_IP
              host.user = env.REMOTE_HOST_USER
              host.password = env.REMOTE_HOST_PWD
              host.allowAnyHosts = 'true'

              sshCommand remote:host, command:"rm -rf ~/openjdk-install"
              sshPut remote:host, from:"./install/openjdk-install", into:"."
              sshCommand remote:host, command:"cd ~/openjdk-install;sh install.sh --install"

              sshCommand remote:host, command:"rm -rf ~/maven-install"
              sshPut remote:host, from:"./install/maven-install", into:"."
              sshCommand remote:host, command:"cd ~/maven-install;sh install.sh --install"

              sshCommand remote:host, command:"source /etc/profile"

              sshPut remote:host, from:"./install/jenkins-install", into:"."
              sshCommand remote:host, command:"cd ~/jenkins-install;sh install.sh --install"
            }
          }
        }
      }
    }
  }
}