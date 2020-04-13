node {
    // Get Artifactory server instance, defined in the Artifactory Plugin administration page.
    def artifactoryServer = Artifactory.server "artifactory"
    // Create an Artifactory Maven instance.
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo
    
    rtMaven.tool = "maven"

    stage('Clone sources') {
        git url: 'https://github.com/richardphiliphaynes/marslander-web-portal.git'
    }

    stage('Artifactory configuration') {
        // Tool name from Jenkins configuration
        rtMaven.tool = "M3"
        // Set Artifactory repositories for dependencies resolution and artifacts deployment.
        rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: artifactoryServer
        //rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: artifactoryServer
    }

    stage('Static code analysis') {
        //sh 'mvn ${env.SONAR_MAVEN_GOAL} -Dsonar.host.url=${env.SONAR_HOST_URL}'
        //withMaven({
        //    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar -Dsonar.host.url=http://40.78.6.96:9000/'
        //})
        def mvn_version = 'M3'
        slackSend(message: "STARTED: Static code analysis of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        try {
            withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
                sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar -Dsonar.host.url=http://40.78.6.96:9000/'
            }
            slackSend(message: "SUCCESSFUL: Static code analysis of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        } catch (err) {
            slackSend(message: "FAILURE: Static code analysis of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        }
    }

    stage('Maven build') {
        slackSend(message: "STARTED: Maven build of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        try {
            buildInfo = rtMaven.run tool: 'M3', pom: 'pom.xml', goals: 'clean install'
            slackSend(message: "SUCCESSFUL: Maven build of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        } catch (err) {
            slackSend(message: "FAILURE: Maven build of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        }
    }

    stage('marslander-pipeline-deploy-to-qa') {
        slackSend(message: "STARTED: QA deployment of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        try {
            deploy(adapters: [[$class: 'Tomcat7xAdapter', url: 'http://3.15.174.32:8080/', credentialsId: 'tomcat']], war: '**/*.war', contextPath: '/QAMarslanderWebPortal')
            slackSend(message: "SUCCESSFUL: QA deployment of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        } catch (err) {
            slackSend(message: "FAILURE: QA deployment of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        }
    }

    stage('Publish build info') {
        slackSend(message: "STARTED: Artifactory deployment of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        try {
            artifactoryServer.publishBuildInfo buildInfo
            slackSend(message: "SUCCESSFUL: Artifactory deployment of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        } catch (err) {
            slackSend(message: "FAILURE: Artifactory deployment of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        }
    }

    stage('marslander-pipeline-deploy-to-production') {
        slackSend(message: "STARTED: Production deployment of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        try {
            deploy(adapters: [[$class: 'Tomcat7xAdapter', url: 'http://3.19.211.54:8080/', credentialsId: 'tomcat']], war: '**/*.war', contextPath: '/ProdMarslanderWebPortal')
            slackSend(message: "SUCCESSFUL: Production deployment of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        } catch (err) {
            slackSend(message: "FAILURE: Production deployment of job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        }
    }

}
