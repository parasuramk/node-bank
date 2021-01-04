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
  stage('Deploy') {
    sshPublisher(
        continueOnError: false, failOnError: true,
        publishers: [
            sshPublisherDesc(
                configName: "appserver2",
                verbose: true,
                transfers: [
                  sshTransfer(execCommand: "docker stop nodebank"),
                  sshTransfer(execCommand: "docker rm nodebank"),
                  sshTransfer(execCommand: "docker pull parasuramk/nodebank:${env.BUILD_NUMBER}"),
                    sshTransfer(execCommand: "docker run -p 3000:3000 -d --name nodebank parasuramk/nodebank:${env.BUILD_NUMBER}")
                ]
            )
        ]
    )
  }
  stage ('Test') {
    echo 'Running functional tests ...'
    catchError (stageResult: 'FAILURE') {
      def rValue = sh (
        script: 'curl -X GET -H \'Content-type: application/json\' -H \'Accept: application/json\' -H \'Authorization: y78x1uG7kfgr00c2\' https://iqe.maveric-systems.com/rapidtest/api/execution/runid/5ff032d7a8ff92f3491723f1',
        returnStdout: true
        )
      echo rValue
      def jsonOutput = readJSON text: rValue
      def schedId = jsonOutput.data.schdlId
      echo "Schedule ID: ${schedId}"
      echo schedId
      
      waitUntil(initialRecurrencePeriod: 15000) {
        rValue = sh (
          script: "curl -X GET -H \'Content-type: application/json\' -H \'Accept: application/json\' -H \'Authorization: y78x1uG7kfgr00c2\' https://iqe.maveric-systems.com/rapidtest/api/execution/status/${schedId}",
          returnStdout: true
        )
        
        jsonOutput = readJSON text: rValue

        if (jsonOutput.data.statusMsg == 'COMPLETED')  // this is a comparison.  It returns true
        {
          echo jsonOutput.summary
          return true
        }
      }
    }
  }
}
