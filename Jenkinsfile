#!/usr/bin/env groovy
import groovy.json.JsonOutput
import groovy.json.JsonSlurper

pipeline {
    agent any
    tools {
        maven 'myMaven'
    }
    stages {
        stage("Determine new version") {
            steps {
                script {
                    env.DEPLOY_VERSION = sh (returnStdout: true, script: "git fetch --tags | git describe --tags `git rev-list --tags --max-count=1`").trim()
                    env.DEPLOY_MAJOR_VERSION = sh (returnStdout: true, script: "echo '${env.DEPLOY_VERSION}' | awk -F'[ .]' '{print \$1}'").trim()
                    env.DEPLOY_COMMIT_HASH = sh (returnStdout: true, script: "git rev-parse HEAD | cut -c1-7").trim()
                    env.DEPLOY_BUILD_DATE = sh (returnStdout: true, script: "date -u +'%Y-%m-%dT%H:%M:%SZ'").trim()
                    env.IS_NEW_VERSION = sh (returnStdout: true, script: "[ '${env.DEPLOY_VERSION}' ] && echo 'YES'").trim()
                    env.COMMITS_ON_MASTER = sh(script: "git rev-list HEAD --count", returnStdout: true).trim()
                }
            }
        }

        stage('Create new version') {
            when {
                environment name: "IS_NEW_VERSION", value: "YES"
            }
            steps {
                script {
                    sh('git remote -v')
                        withCredentials([usernamePassword(credentialsId: 'username-with-password', 
                         passwordVariable: 'GIT_PASS', usernameVariable: 'GIT_USER')]) {
                        sh('git config --global user.email thegioiitjob@gmail.com')
                        sh('git config --global user.name xuqupro')
                        def versionType="MAJOR"
                        def nversion=getTagVersion(versionType)
                        echo("${nversion}")
                       
                        // sh('git tag -a "v${DEPLOY_VERSION}" -m "Job: pipeline_no1"')
                        // sh('git push https://$GIT_USER:$GIT_PASS@github.com/xuqupro/helloworld.git --tags')
                       // sh('git push origin 1.9')
                    }
                }
            }
        }
        stage('notification') {
            steps {
                notification()
            }
        }
        stage('Build') {
            steps {
                buildpr()
            }
        }
    }
}
def checkout(){
    checkout scm
}

def notification() {
    def DATE = sh(returnStdout: true, script: "date +'%d-%m-%y'")
    def API_A = "https://hooks.slack.com/services/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"
    def NOTIFICATION_SUCCESS = "'{\"text\":\"Hello World!, ${DATE}\"}'"
    sh "curl -X POST -H 'Content-type: application/json' --data ${NOTIFICATION_SUCCESS} ${API_A}"
}

def buildpr() {
    try {
        echo "\u2600 BUILD_URL=${env.BUILD_URL}"
        def workspace = pwd()
        echo "\u2600 workspace=${workspace}"
    }catch(exc){
        currentBuild.result = "FAILURE"
    }
}

def getTagVersion(versionType) {
    // def tag=shell(returnStdout: true, script: 'git tag --sort version:refname | tail -1').trim()
    def tag=sh (returnStdout: true, script: "git fetch --tags | git describe --tags `git rev-list --tags --max-count=1`").trim()
    if (tag==null || tag.size() == 0) {
        echo "No existing tag found. Using version: ${version}"
        return "0.1"
    }
    tag=tag.trim()
    double rate=0.0
    switch(versionType){
        case "MAJOR":
            rate = (Double.parseDouble("${tag}") + Double.parseDouble("1.0"))
            break
        case "MINOR":
            rate = (Double.parseDouble("${tag}") + Double.parseDouble("0.1"))
            break
        default :
            rate = (Double.parseDouble("${tag}") + Double.parseDouble("1.0"))
            break
    }
    double version_auto =  Math.round(rate * 10) / 10 
    return version_auto
}