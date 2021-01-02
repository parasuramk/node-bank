node {
  stage ('SonarQube Code Analysis') {
    echo 'Code analysis ...'
    withSonarQubeEnv('sonarqube') {
      sh '''/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonarqube/bin/sonar-scanner -Dsonar.host.url=http://34.126.117.238:9000 -Dsonar.login=admin -Dsonar.password=Passw0rd -Dsonar.projectKey=nodebank -Dsonar.source=.
'''
    }
  }
  stage ('Build') {
    echo 'Building docker image'
    checkout scm
    docker.withRegistry('https://registry.hub.docker.com', '9abfe119-8a8d-41a6-b888-1093b34aa3f6') {

        def customImage = docker.build("parasuramk/nodebank:${env.BUILD_NUMBER}")

        /* Push the container to the custom Registry */
        customImage.push()
    }
  }
}
