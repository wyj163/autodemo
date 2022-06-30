def APP_ENV = "dev"
def PATH_IPA_FILE = ""
def currentDate=""
def PATH_PLIST = ""
def SCHEME = ""
def BUNDLE_ID = "com.infinitus.bupm.dev"
def PATH_PROVISIONING_PROFILE = "gbss dev"
def ROBOT_ADDRESS = "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=e00b4393-50e5-47be-b682-907763f03d88"
//小飞机
def API_TOKEN = "cfe2c448ada86695395f5763943b62b2"
//蒲公英
def UKEY = "f3e9556ecb34187ad5c3e014fa2465c6"
def API_KEY = "4cc441bf79f4ecfc91fa35acbde1d6f4"
def APP_NAME = "无限极开发版"
def CHANNEL_TOKEN = "caca04445d6b4c72a62a8f679a16d1c1"
def FIR_PATH = "znwg"
def appQRCodeURL = ""
def appShortcutURL = ""

import java.text.SimpleDateFormat
import groovy.json.JsonSlurperClassic


@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}

@NonCPS
def convertToMarkdown(commitMessage) {
    echo "convertToMarkdown() - origin: ${commitMessage}"
    if (!commitMessage) {
//      commitMessage = "fix(MAPIGW-1234, MAPIGW-1223): xxxwwwyyy"
        return "";
    }
    def jiraIdPattern = ~"\\w{4,9}\\-\\d{2,6}";
    def matcher = (commitMessage =~ jiraIdPattern);

    echo "convertToMarkdown() - matcher: ${matcher}"
    if (!matcher) {
        return commitMessage;
    }
    def index = commitMessage.indexOf(matcher[0])
    commitMessage = commitMessage.substring(0, index) + "[" + matcher[0] + "](https://jira.infinitus.com.cn/browse/" + matcher[0] + ")" + commitMessage.substring(index+matcher[0].length(), commitMessage.length());
    echo "convertToMarkdown() - result: ${commitMessage}"
    return commitMessage
}

def getCommitLog(){
    def changeString = ""

    echo "Gathering SCM changes"
    def changeLogSets = currentBuild.changeSets
    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items
        for (int j = 0; j < entries.length; j++) {
            def entry = convertToMarkdown(entries[j].msg)
            changeString += " ${j + 1}. ${entry} [${entries[j].author}]\\n"
        }
    }
    echo "封装完成"
    if (!changeString) {
        changeString = "没更新"
    }
    // 去掉双引号防止shell命令行脚本异常
    changeString = changeString.replaceAll("\"", " ")
    echo "最终的结果：" + changeString
    return changeString
}

def sendMsgToRobot(msg, robotUrl = "") {
    if (robotUrl == "") {
        robotUrl = "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=6a5a9f9e-3fce-411c-9944-6f1b3cd0bf4c"
    }
    sendShellScipt = """
        curl '${robotUrl}' \
        -H 'Content-Type: application/json' \
        -d '
        {
            "msgtype": "text",
            "text": {
                "content": "${msg}"
            }
        }'
    """
    sh sendShellScipt
}

def sendSuccessMsg(msg, appShortcutURL, appQRCodeURL, robotUrl = ""){
    if(robotUrl == ""){
        robotUrl = "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=6a5a9f9e-3fce-411c-9944-6f1b3cd0bf4c"
    }
        sendShellScipt1 = """
        curl '${robotUrl}' \
        -H 'Content-Type: application/json' \
        -d '
        {
            "msgtype": "markdown",
            "markdown": {
                "content": "<font color='green'>${msg}</font>
                    ><font color='blue'>下载地址：</font>[${appShortcutURL}](${appShortcutURL})
                    >[二维码下载](${appQRCodeURL})
                "
            }
        }'
    """
    sh sendShellScipt1
}

pipeline {
    agent any

    environment{
        PATH_WWW = "${env.WORKSPACE}/App/App/Resource/www/"
        PATH_SAVE = "/Users/devops/archive/大平台3.0/"
        PATH_EXPORT_PLIST = "${env.WORKSPACE}/ExportOptions.plist"
    }

    parameters {
        booleanParam(name: 'uploadToFir', defaultValue: false, description: '是否上传到小飞机')
        booleanParam(name: 'uploadToPgyer', defaultValue: true, description: '是否上传到蒲公英')
        booleanParam(name: 'notifyToWorkWX', defaultValue: true, description: '是否通知到群')
    }
    
    stages {
        stage('Copy Resource') {
            steps {
                script {
                    env.GIT_COMMIT_MSG = sh(script: 'git log -1 --pretty=%B ${GIT_COMMIT}', returnStdout: true).trim()
                    //sendMsgToRobot("${env.JOB_NAME}开始发布!\\n发布内容:\\n${env.GIT_COMMIT_MSG}")
                    //sendMsgToRobot("${env.JOB_NAME}开始发布!\\n发布内容:\\n${env.GIT_COMMIT_MSG}","${ROBOT_ADDRESS}")
                    def dateFormat = new SimpleDateFormat("yyyy-MM-dd")
    			    def date = new Date()
    			    currentDate=dateFormat.format(date)
                    if(env.BRANCH_NAME == 'develop'){
                        APP_ENV = sh(returnStdout: true, script: 'echo staging').trim()
                        SCHEME = sh(returnStdout: true, script: 'echo Staging').trim()
                        PATH_PLIST = "${env.WORKSPACE}/App/App/Supporting Files/App-${SCHEME}.plist"
                        BUNDLE_ID = "com.infinitus.bupm.staging"
                        PATH_PROVISIONING_PROFILE = "gbss staging"
                        API_TOKEN = "178135da943343727c4eaabbc82767a6"
                        CHANNEL_TOKEN = "5ca43eafbe0843fe9f856507dd5cf6fb"
                        APP_NAME = "员工体验版"
                        FIR_PATH = "znwg"

                    }

                    if(env.BRANCH_NAME == 'feature/devint'){
                        APP_ENV = sh(returnStdout: true, script: 'echo dev').trim()
                        SCHEME = sh(returnStdout: true, script: 'echo Dev').trim()
                        PATH_PLIST = "${env.WORKSPACE}/App/App/Supporting Files/App-${SCHEME}.plist"
                        BUNDLE_ID = "com.infinitus.bupm.dev"
                        PATH_PROVISIONING_PROFILE = "gbss dev"
                        API_TOKEN = "cfe2c448ada86695395f5763943b62b2"
                        CHANNEL_TOKEN = "caca04445d6b4c72a62a8f679a16d1c1"
                        APP_NAME = "无限极开发版"
                        FIR_PATH = "bupmidev"

                    }

                    if(env.BRANCH_NAME == 'master'){
                        APP_ENV = sh(returnStdout: true, script: 'echo release').trim()
                        PATH_PLIST = "${env.WORKSPACE}/App/App/Supporting Files/App.plist"
                        BUNDLE_ID = "com.infinitus.bupm"
                        PATH_PROVISIONING_PROFILE = "distributionbupm"
                        API_TOKEN = "178135da943343727c4eaabbc82767a6"
                        CHANNEL_TOKEN = "6786f7cc4bd44621b136b8a05a323a23"
                        APP_NAME = "无限极"
                        FIR_PATH = "znwg"
                    }
                    PATH_IPA_FILE = "${PATH_SAVE}${APP_ENV}/${currentDate}/gbss-ios.${APP_ENV}.v3.0.${BUILD_NUMBER}.ipa"
                    if(params.notifyToWorkWX){
                        sendMsgToRobot("【" + "${APP_NAME}(iOS)-v3.0.${BUILD_NUMBER}" + "】" + "开始发布, 内容:\\n" + getCommitLog(),"${ROBOT_ADDRESS}")
                    }
                }
                sh """
                    mkdir -p ${PATH_SAVE}${APP_ENV}/${currentDate}
                    cp -R ~/wwwzip/${APP_ENV}/cenarius-config.json ${PATH_WWW}
                    cp -R ~/wwwzip/${APP_ENV}/cenarius-routes.json ${PATH_WWW}
                    cp -R ~/wwwzip/${APP_ENV}/www.zip ${PATH_WWW}
                    cd ${PATH_WWW}
                    zip -r www.zip cordova cordova-js-src
                    /usr/libexec/PlistBuddy -c "Set :CFBundleVersion 3.0.${BUILD_NUMBER}" "${PATH_PLIST}"
                    /usr/libexec/PlistBuddy -c "Set :CFBundleShortVersionString 3.0.${BUILD_NUMBER}" "${PATH_PLIST}"
                """
            }
        }
        
        stage('Build'){
            steps {
                sh """
                    xcodebuild clean -workspace gbss-ios-app.xcworkspace -scheme App-${SCHEME} -configuration RELEASE
                    xcodebuild archive -workspace gbss-ios-app.xcworkspace -scheme App-${SCHEME} -configuration RELEASE -sdk iphoneos -archivePath ArchivePath
                    pwd
                    rm -f ${PATH_EXPORT_PLIST}
                    /usr/libexec/PlistBuddy -c  "Add :method String ad-hoc"  ${PATH_EXPORT_PLIST}
                    /usr/libexec/PlistBuddy -c  "Add :destination String export"  ${PATH_EXPORT_PLIST}
                    /usr/libexec/PlistBuddy -c  "Add :signingCertificate String Apple Distribution"  ${PATH_EXPORT_PLIST}
                    /usr/libexec/PlistBuddy -c  "Add :stripSwiftSymbols bool true"  ${PATH_EXPORT_PLIST}
                    /usr/libexec/PlistBuddy -c  "Add :signingStyle String manual"  ${PATH_EXPORT_PLIST}
                    /usr/libexec/PlistBuddy -c  "Add :compileBitcode bool true"  ${PATH_EXPORT_PLIST}
                    /usr/libexec/PlistBuddy -c  "Add :teamID String Q92H77L2WN"  ${PATH_EXPORT_PLIST}
                    /usr/libexec/PlistBuddy -c  "Add :thinning String <none>"  ${PATH_EXPORT_PLIST}
                    /usr/libexec/PlistBuddy -c  "Add :provisioningProfiles dict"  ${PATH_EXPORT_PLIST}
                    /usr/libexec/PlistBuddy -c  "Add :provisioningProfiles:${BUNDLE_ID} String ${PATH_PROVISIONING_PROFILE}"  ${PATH_EXPORT_PLIST}
                    xcodebuild -exportArchive -exportOptionsPlist ${PATH_EXPORT_PLIST} -archivePath ArchivePath.xcarchive -exportPath ${env.WORKSPACE}/autoArchive  -allowProvisioningUpdates
                    mv ${env.WORKSPACE}/autoArchive/App-${SCHEME}.ipa ${PATH_IPA_FILE}
                    rm -rf ${env.WORKSPACE}/autoArchive
                    rm -f ${PATH_EXPORT_PLIST}
                    rm -rf ${env.WORKSPACE}/ArchivePath.xcarchive
                """
            }
        }

        stage('Publish Pgyer'){
            when {
                not {
                    branch 'master'
                }
                expression{
                    return params.uploadToPgyer
                }
            }

            steps{
                script {
                    def response2 = sh returnStdout: true, script:
                    """
                    curl -F "file=@${PATH_IPA_FILE}" -F "_api_key=${API_KEY}" -F "buildInstallType=2" -F "buildPassword=Infinitus2021!" https://www.pgyer.com/apiv2/app/upload
                    """
                    println "response2: " + response2
                    states = jsonParse(response2)
                    if(states != null && states['data']){
                        appQRCodeURL = states['data']['buildQRCodeURL']
                        appShortcutURL = "https://pgyer.com/" + states['data']['buildShortcutUrl']
                        def buildKey = states['data']['buildKey']

                        sh """
                        curl "https://channel-${APP_ENV}.infinitus.com.cn/admin/mobile/appversions/add/staging/${CHANNEL_TOKEN}" -X "POST" -F "appVerVo.platformid=iPhone" -F "appVerVo.status=1" -F "appVerVo.identifier=${BUNDLE_ID}" -F "appVerVo.appName=${APP_NAME}" -F "appVerVo.version=3.0.${BUILD_NUMBER}" -F "appVerVo.build=30${BUILD_NUMBER}" -F "appVerVo.releaseNote=感谢使用${APP_NAME}！为了让你获得更加理想的体验，我们会定期更新应用。" -F "appVerVo.packageSize=90" -F "appVerVo.forceUpdate=0" -F "appVerVo.installPackType=1" -F "appVerVo.plist=itms-services://?action=download-manifest&url=https://www.pgyer.com/app/plist/${buildKey}" -F "appVerVo.useBrowser=1" -F "appVerVo.autoPublish=0"
                        """
                    }
                }

            }

            post{
                success{
                    script{
                       if(params.notifyToWorkWX){
                            //sendSuccessMsg("【${APP_NAME}-Android-V3.0.${BUILD_NUMBER}】发布完成!","${appShortcutURL}","${appQRCodeURL}")
                            sendSuccessMsg("【${APP_NAME}(iOS)-v3.0.${BUILD_NUMBER}】发布完成!","${appShortcutURL}","${appQRCodeURL}","${ROBOT_ADDRESS}")
                        }
                    }
                }
            }
        }

        stage('Publish'){
            when{
                not{
                    branch 'master'
                }
                expression{
                    return params.uploadToFir
                }
            }
            steps {
                script {
                   def response2 = sh returnStdout: true, script:
                   """curl -X "POST" "http://api.bq04.com/apps" -H "Content-Type: application/json" -d '{"type":"ios", "bundle_id":"${BUNDLE_ID}", "api_token":"${API_TOKEN}"}'
                   """
                   println "response2: " + response2
                   states = jsonParse(response2)
                   if(states != null && states['cert'] != null && states['cert']['binary']){
                       def upload_key = states['cert']['binary']['key']
                       def upload_token = states['cert']['binary']['token']
                       println "key: " + upload_key
                       println "token: " + upload_token

                       sh """
                       curl -F "key=${upload_key}" -F "token=${upload_token}" -F "file=@${PATH_IPA_FILE}" -F "x:name=${APP_NAME}" -F "x:version=3.0.${BUILD_NUMBER}" -F "x:build=${BUILD_NUMBER}" https://up.qbox.me --http1.1
                       curl "https://channel-${APP_ENV}.infinitus.com.cn/admin/mobile/appversions/add/staging/${CHANNEL_TOKEN}" -X "POST" -F "appVerVo.platformid=iPhone" -F "appVerVo.status=1" -F "appVerVo.identifier=${BUNDLE_ID}" -F "appVerVo.appName=${APP_NAME}" -F "appVerVo.version=3.0.${BUILD_NUMBER}" -F "appVerVo.build=30${BUILD_NUMBER}" -F "appVerVo.releaseNote=感谢使用${APP_NAME}！为了让你获得更加理想的体验，我们会定期更新应用。" -F "appVerVo.packageSize=90" -F "appVerVo.forceUpdate=0" -F "appVerVo.installPackType=1" -F "appVerVo.plist=http://bupappdownload-${APP_ENV}.infinitus.com.cn/${FIR_PATH}" -F "appVerVo.useBrowser=1" -F "appVerVo.autoPublish=0"
                       """
                   }
               }
            }
            post{
                success{
                    script{
                        if(params.notifyToWorkWX){
                            //sendMsgToRobot("【${env.JOB_NAME}】发布完成!\\n下载地址:http://bupappdownload-${APP_ENV}.infinitus.com.cn/${FIR_PATH}")
                            sendMsgToRobot("【${APP_NAME}(Android)-V3.0.${BUILD_NUMBER}】发布完成!\\n下载地址:http://bupappdownload-${APP_ENV}.infinitus.com.cn/${FIR_PATH}", "${ROBOT_ADDRESS}")
                        }
                    }
                }
            }
        }
    }
    post {
        failure{
            script{
                   if(params.notifyToWorkWX){
                    //sendMsgToRobot("【${env.JOB_NAME}】发布失败,请及时处理!!!")
                    sendMsgToRobot("【${APP_NAME}(iOS)-V3.0.${BUILD_NUMBER}】发布失败,请及时处理!!!", "${ROBOT_ADDRESS}")
                }
            }
        }

        aborted{
            script{
                if(params.notifyToWorkWX){
                    //sendMsgToRobot("【${env.JOB_NAME}】发布中断")
                    sendMsgToRobot("【${APP_NAME}(iOS)-V3.0.${BUILD_NUMBER}】发布中断", "${ROBOT_ADDRESS}")
                }
            }
        }
    }
}
