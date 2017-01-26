node {
    setupEnv()

    try {
        stage 'Prepare'
        checkout scm

        sh "chmod +x ./mvnw"
        sh "./mvnw org.apache.maven.plugins:maven-help-plugin:evaluate -Dexpression=project.version | grep -v '\\[' > .version"
        def version = readFile('.version').toString().trim()

        echo "Starting pipeline for version ${version}_${env.BUILD_NUMBER}"

        lock('LockableThing') {
            stage 'Build'
            sh './mvnw -T 1C clean install -pl \\!test/functional-tests -Dskip.coverage=false'
            step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml,**/target/failsafe-reports/TEST-*.xml'])

            stage 'Sonar Analysis'
            echo "DO SONAR ANALYSIS"

            stage 'Functional Test'
            echo "RUN FUNCTIONAL TESTS"

            stage 'Push to Nexus'
            echo "PUSH TO NEXUS"
        }

        stage 'Configure Consul'
        echo  "CONFIGURING CONSUL"

        lock('AWS-DEV') {
            stage 'Deploy to AWS'
            echo "DEPLOY TO AWS"

            stage 'Deployment Health Check'
            echo "DEPLOYMENT HEALTH CHECK"
        }
    }
    catch(e) {
//        hipchatSend(color: 'RED',
//                notify: true,
//                message:  "Jenkins Job Failed: <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>")
        throw e
    }
}

void setupEnv() {
    env.JAVA_HOME = "/usr/lib/jvm/java-8-openjdk-amd64"
    env.PATH = "/usr/local/bin:${env.JAVA_HOME}/bin:${env.PATH}"
}
