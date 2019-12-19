timeout ([time: 10, unit: 'MINUTES']) {

def label = "jenkins-slave-${UUID.randomUUID().toString()}"

properties([
    buildDiscarder(
        logRotator(
            numToKeepStr: '30'
        )
    )
])

podTemplate(label: label, yaml: '''

spec:
  containers:
    - name: jnlp
''', containers: [
        containerTemplate(
            name: 'jnlp',
            image: 'jenkins/jnlp-slave:latest',
            alwaysPullImage: true,
            args: '${computer.jnlpmac} ${computer.name}',
        ),

        containerTemplate(
            name: 'docker-dind',
            image: 'docker:19-dind',
            alwaysPullImage: true,
            privileged: true,
            envVars: [
                envVar(key: 'DOCKER_TLS_CERTDIR', value: '')
            ],
        ),

        containerTemplate(
            name: 'docker',
            image: 'docker:19',
            alwaysPullImage: true,
            ttyEnabled: true,
            command: 'cat',
            envVars: [
                envVar(key: 'DOCKER_HOST', value: 'tcp://localhost:2375')
            ],
        ),
    ]) {

    node(label) {
        stage('clone') {
            checkout scm
        }

        stage('build') {
            container('docker') {
                def tag = "${env.JOB_NAME}:${env.BUILD_ID}".toLowerCase()

                devTools = docker.build(tag, "--pull -f .docker/dev-tools/Dockerfile .")

                devTools.inside() {
                    sh 'whoami'
                }
            }
        }
    }
}

}
