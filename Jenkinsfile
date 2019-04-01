#!/usr/bin/env groovy
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
                    env.TAG = sh(returnStdout: true, script: 'git tag --sort version:refname | tail -1').trim()
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
                        def nversion=getTagVersion()
                        echo("${nversion}")
                        echo("${env.DEPLOY_MAJOR_VERSION}")
                        echo("${env.COMMITS_ON_MASTER}")
                        // sh('git tag -a "v${DEPLOY_VERSION}" -m "Job: pipeline_no1"')
                        // sh('git push https://$GIT_USER:$GIT_PASS@github.com/xuqupro/helloworld.git --tags')
                       // sh('git push origin 1.9')
                    }
                }
            }
        }
    }
}
def getTagVersion() {
    // def tag=shell(returnStdout: true, script: 'git tag --sort version:refname | tail -1').trim()
    def tag=sh (returnStdout: true, script: "git fetch --tags | git describe --tags `git rev-list --tags --max-count=1`").trim()
    if (tag==null || tag.size() == 0) {
        echo "No existing tag found. Using version: ${version}"
        return "0.1"
    }
    tag=tag.trim()
    sh "git fetch"

    def version_curent = sh(returnStdout: true, script: "git tag")
    def l = (ArrayList<String>) version_curent.split()

    if (l.size() == 0) {
        l.add("v0.0")
    } 

    def beforeColon = l.pop().substring(1)
    double rate = (Double.parseDouble("${beforeColon}") + Double.parseDouble("0.1"))
    double version_auto =  Math.round(rate * 10) / 10 
    return version_auto
}