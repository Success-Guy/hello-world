pipeline {
    agent any
    environment {
        DOCKER_HOST = '3.17.5.181'
    }

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven 'M2'
    }

    stages {
        stage('Building...') {
      steps {
        // Get some code from a GitHub repository
        git branch: 'ansible', credentialsId: 'github-private-key', url: 'git@github.com:Success-Guy/hello-world.git'
       
      //build maven
      sh 'mvn clean package'
      }
        }
        stage('Deplying to Docker server') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'ansible-server', passwordVariable: 'pwd', usernameVariable: 'user')]) {
            remote = [:]
            remote.name = "${ user }"
            remote.host = "${ DOCKER_HOST }"
            remote.user = "${ user }"
            remote.password = "${ pwd }"
            remote.allowAnyHosts = true

            sshPut remote: remote, from: 'webapp/target/webapp.war', into: '.'
            sshPut remote: remote, from: 'Dockerfile', into: '.'
	    sshPut remote: remote, from: 'ansible-playbook.yml', into:'.'
            //this will only work on the first run, else fail due to duplicate docker name
            //else ansible will be integrated in the next step
            sshCommand remote: remote, command: "mv webapp.war ROOT.war"
            sshCommand remote: remote, command: "ansible-playbook -i hosts ansible-playbook.yml"
          }
        }
        echo 'Done succesfully'
      }
        }
    }
}
