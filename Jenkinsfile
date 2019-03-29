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
                        setTags(env.DEPLOY_VERSION)
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

def setTags(tag){
    double converted=Double.parseDouble(tag);
    echo("${converted}")
}