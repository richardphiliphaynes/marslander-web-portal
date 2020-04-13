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
        rtMaven.tool = "maven"
        // Set Artifactory repositories for dependencies resolution and artifacts deployment.
        rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: artifactoryServer
        rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: artifactoryServer
    }

    stage('Static code analysis') {
        sh 'mvn ${env.SONAR_MAVEN_GOAL} -Dsonar.host.url=${env.SONAR_HOST_URL}'
    }

    stage('Maven build') {
        buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
    }

    //stage('marslander-pipeline-deploy-to-qa') {
    //    deploy adapters: "tomcat7" war: "**/*.war" contextPath: "/QAWebapp" url: "http://3.15.174.32:8080/" credentialsId: "tomcat"
    //}

    stage('Publish build info') {
        artifactoryServer.publishBuildInfo buildInfo
    }

    //stage('marslander-pipeline-deploy-to-production') {
    //    deploy adapters: "tomcat7" war: "**/*.war" contextPath: "/QAWebapp" url: "http://3.19.211.54:8080/" credentialsId: "tomcat"
    //}

}
