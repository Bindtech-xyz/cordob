import groovy.json.JsonOutput
import static java.util.UUID.randomUUID
def uuid = randomUUID() as String
def sublabel = uuid.take(10)

def author = "";

def getGitAuthor = {
    def commit = sh(returnStdout: true, script: 'git rev-parse HEAD')
    author = sh(returnStdout: true, script: "git --no-pager show -s --format='%an' ${commit}").trim()
}
def message = "";

def getLastCommitMessage = {
    message = sh(returnStdout: true, script: 'git log -1 --pretty=%B').trim()
}

def slackNotificationChannel = 'jenkins'     // ex: = "builds"

def notifySlack(text, channel, attachments) {
    def slackURL = 'https://hooks.slack.com/services/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'
    def jenkinsIcon = 'https://wiki.jenkins-ci.org/download/attachments/2916393/logo.png'

    def payload = JsonOutput.toJson([text: text,
        channel: channel,
        username: "Jenkins",
        icon_url: jenkinsIcon,
        attachments: attachments
    ])

    sh "curl -X POST --data-urlencode \'payload=${payload}\' ${slackURL}"
}

podTemplate(containers: [
    containerTemplate(
        name: 'node', 
        //image: 'node:14-alpine', 
        image: 'docker.dev.live-arts.com/d3n/node-android:latest',
        ttyEnabled: true,
        alwaysPullImage: true,
        command: 'cat'
    ),
    ],
    imagePullSecrets: [ 'regsecret' ],
    volumes: [
        configMapVolume(mountPath: '/root/.docker', configMapName: 'docker-config'),
        configMapVolume(mountPath: '/root/.m2', configMapName: 'm2settings'),
        //persistentVolumeClaim(mountPath: '/nodeModules/node_modules', claimName: 'maven-repo', readOnly: false)
        //persistentVolumeClaim(mountPath: '/root/.npm', claimName: 'maven-repo', readOnly: false)
    ]) 
    {
            node(POD_LABEL) {
            def namespace = "devops-tools"
            
            stage('Checkout') {
                checkout()    
            //    def myRepo = checkout([
            //        $class: 'GitSCM', branches: [[name: '*/dev']], 
            //        doGenerateSubmoduleConfigurations: false, 
            //        extensions: [], submoduleCfg: [], 
            //        userRemoteConfigs: [[
            //            credentialsId: 'live-arts-github',
            //            url: 'https://github.com/live-arts/livearts-mobile.git'
            //        ]]
            //    ])
            //    env.git_Commit = myRepo.GIT_COMMIT
            //    env.git_Branch = myRepo.GIT_BRANCH
            //    env.short_Commit = "${env.git_Commit[0..7]}"
            //    //env.NODE_PATH = "/nodeModules/node_modules"
            
            }
            stage('Install git') {
                container('node') {
                    print "Install git"
                    print "MESSAGE = ${message}"
                    print "AUTHOR  = ${author}"
                    echo "The branch is ${env.GIT_BRANCH}."                    
                    print "JOB NUMBER ${env.JOB_NAME}"
                    print "BUILD NUMBER ${env.BUILD_NUMBER}"
                    print "BUILD URL ${env.BUILD_URL}"
                    notifySlack("Success!", slackNotificationChannel, [])
                }
            }
        }
    }
