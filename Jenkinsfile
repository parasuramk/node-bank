node {
  stage ('SonarQube Code Analysis') {
    echo 'Code analysis ...'
    withSonarQubeEnv('sonarqube') {
      sh '''/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonarqube/bin/sonar-scanner -Dsonar.host.url=http://34.126.117.238:9000 -Dsonar.login=70b003e24882bf6212546b97ecf26d4771d03548 -Dsonar.projectKey=NodeBank -Dsonar.source=.
'''
    }
  }
  stage ('SonarQube Quality Gate') {
    echo 'Quality Gate ...'
    timeout(time: 30, unit: 'MINUTES') { 
      def qg = waitForQualityGate(abortPipeline: true, credentialsId: 'sonarlogin') // Reuse taskId previously collected by withSonarQubeEnv
      if (qg.status != 'OK') {
          error "Pipeline aborted due to quality gate failure: ${qg.status}"
      }
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
  
  stage ('Build Verification') {
    echo 'Running sanity tests ...'
    catchError (stageResult: 'FAILURE') {
      def rValue = sh (
        script: 'curl -X GET -H \'Content-type: application/json\' -H \'Accept: application/json\' -H \'Authorization: y78x1uG7kfgr00c2\' https://iqe.maveric-systems.com/rapidtest/api/execution/runid/5ff3ee1614590e6225f36192',
        returnStdout: true
        )

      def jsonOutput = readJSON text: rValue
      def schedId = jsonOutput.data.schdlId
      
      timeout(time: 10, unit: 'MINUTES') {
        waitUntil {
          script {
            rValue = sh (
              script: "curl -X GET -H \'Content-type: application/json\' -H \'Accept: application/json\' -H \'Authorization: y78x1uG7kfgr00c2\' https://iqe.maveric-systems.com/rapidtest/api/execution/status/${schedId}",
              returnStdout: true
            )

            jsonOutput = readJSON text: rValue
            echo jsonOutput.data.statusMsg
            
            if (jsonOutput.data.statusMsg == 'COMPLETED')  // this is a comparison.  It returns true
            {
              return true
            }
            return false
          }
        }
        if (jsonOutput.data.statusMsg == 'COMPLETED')  // this is a comparison.  It returns true
        {
          echo "Total tests executed: ${jsonOutput.data.summary.TOTAL}; Passed: ${jsonOutput.data.summary.PASSED}; Failed: ${jsonOutput.data.summary.FAILED}; Skipped: ${jsonOutput.data.summary.SKIPPED}"
          if (jsonOutput.data.summary.PASSED == jsonOutput.data.summary.TOTAL) {
            //do nothing
            echo 'Sanity tests passed'
          }
          else {
            sh "exit 1"
          }
        }
      }
    }    
  }
  
  stage ('Functional Tests') {
    echo 'Running functional tests ...'
    catchError (stageResult: 'FAILURE') {
      def retValue = sh (
        script: 'curl -X GET -H \'Content-type: application/json\' -H \'Accept: application/json\' -H \'Authorization: y78x1uG7kfgr00c2\' https://iqe.maveric-systems.com/rapidtest/api/execution/runid/5ff3ec691d101261ffddd52d',
        returnStdout: true
        )

      def jsonOutput = readJSON text: retValue
      def schedId = jsonOutput.data.schdlId
      
      timeout(time: 10, unit: 'MINUTES') {
        waitUntil {
          script {
            retValue = sh (
              script: "curl -X GET -H \'Content-type: application/json\' -H \'Accept: application/json\' -H \'Authorization: y78x1uG7kfgr00c2\' https://iqe.maveric-systems.com/rapidtest/api/execution/status/${schedId}",
              returnStdout: true
            )

            jsonOutput = readJSON text: retValue
            echo jsonOutput.data.statusMsg
            
            if (jsonOutput.data.statusMsg == 'COMPLETED')  // this is a comparison.  It returns true
            {
              return true
            }
            return false
          }
        }
        if (jsonOutput.data.statusMsg == 'COMPLETED')  // this is a comparison.  It returns true
        {
          echo "Total tests executed: ${jsonOutput.data.summary.TOTAL}; Passed: ${jsonOutput.data.summary.PASSED}; Failed: ${jsonOutput.data.summary.FAILED}; Skipped: ${jsonOutput.data.summary.SKIPPED}"
          if (jsonOutput.data.summary.PASSED == jsonOutput.data.summary.TOTAL) {
            //do nothing
            echo 'Functional tests passed'
          }
          else {
            sh "exit 1"
          }
        }
      }
    }
  }
}
