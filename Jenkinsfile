properties([
    buildDiscarder(logRotator(numToKeepStr: '5'))
])

timestamps() {
    timeout(time: 10, unit: 'MINUTES') {
        env.ARTIFACT = "${env.JOB_NAME.split('/')[0]}-hello"
        env.REPO_URL = 'https://artifactory.puzzle.ch/artifactory/ext-release-local'
        node { // with hosted env use node(env.JOB_NAME.split('/')[0])
            withCredentials([file(credentialsId: 'm2_settings', variable: 'M2_SETTINGS'), usernameColonPassword(credentialsId: 'jenkins-artifactory', variable: 'ARTIFACTORY'), file(credentialsId: 'known_hosts', variable: 'KNOWN_HOSTS')]) {  // Credentials Binding Plugin
                withEnv(["JAVA_HOME=${tool 'jdk11'}", "PATH+MAVEN=${tool 'maven36'}/bin:${env.JAVA_HOME}/bin"]) {
                    stage('Build') {
                        checkout scm
                        sh 'mvn -B -V -U -e clean verify -Dsurefire.useFile=false -DargLine="-Djdk.net.URLClassPath.disableClassPathURLCheck=true"'
                        sh "mvn -s '${M2_SETTINGS}' -B deploy:deploy-file -DrepositoryId='puzzle-releases' -Durl='${REPO_URL}' -DgroupId='com.puzzleitc.jenkins-techlab' -DartifactId='${ARTIFACT}' -Dversion='1.0' -Dpackaging='jar' -Dfile=`echo target/*.jar`"
                        sshagent(['testserver']) {  // SSH Agent Plugin
                            sh "ls -l target"
                            sh "ssh -o UserKnownHostsFile='${KNOWN_HOSTS}' -p 2222 richard@testserver.vcap.me 'curl -O -u \'${ARTIFACTORY}\' ${REPO_URL}/com/puzzleitc/jenkins-techlab/${ARTIFACT}/1.0/${ARTIFACT}-1.0.jar && ls -l'"
                        }
                        archiveArtifacts 'target/*.?ar'
                        junit 'target/**/*.xml'  // Requires JUnit plugin
                    }
                }
            }
        }
    }
}
