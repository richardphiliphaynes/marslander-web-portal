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
        slackSend(message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        try {
            withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
                sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar -Dsonar.host.url=http://40.78.6.96:9000/'
            }
            slackSend(message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        } catch (err) {
            slackSend(message: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }

    stage('Maven build') {
        buildInfo = rtMaven.run tool: 'M3', pom: 'pom.xml', goals: 'clean install'
    }

    stage('marslander-pipeline-deploy-to-qa') {
        deploy(adapters: [[$class: 'Tomcat7xAdapter', url: 'http://3.15.174.32:8080/', credentialsId: 'tomcat']], war: '**/*.war', contextPath: '/QAMarslanderWebPortal')
    }

    stage('Publish build info') {
        artifactoryServer.publishBuildInfo buildInfo
    }

    stage('marslander-pipeline-deploy-to-production') {
        deploy(adapters: [[$class: 'Tomcat7xAdapter', url: 'http://3.19.211.54:8080/', credentialsId: 'tomcat']], war: '**/*.war', contextPath: '/ProdMarslanderWebPortal')
    }

}
