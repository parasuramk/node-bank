node {
  stage ('build') {
    checkout scm
    docker.withRegistry('https://registry.hub.docker.com', '9abfe119-8a8d-41a6-b888-1093b34aa3f6') {

        def customImage = docker.build("parasuramk/nodebank:${env.BUILD_NUMBER}")

        /* Push the container to the custom Registry */
        customImage.push()
    }
  }
}
