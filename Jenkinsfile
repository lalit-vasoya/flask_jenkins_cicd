pipeline {
    agent any
    stages {
        stage("BUILD") {
            steps {
                echo "BRANCH_NAME=${env.BRANCH_NAME} : GIT_BRANCH=${env.GIT_BRANCH} : GIT_LOCAL_BRANCH=${env.GIT_LOCAL_BRANCH} : CHANGE_ID=${env.CHANGE_ID} : CHANGE_URL=${env.CHANGE_URL} : CHANGE_AUTHER=${env.CHANGE_AUTHER} : CHANGE_TITLE=${env.CHANGE_TITLE}"
                sh '/script/build.sh'
            }
        }
        stage("CHECK-linting") {
            steps {
                echo "Linting"
                sh '/script/check_linting.sh'
            }
        }
        stage("TEST"){
            steps {
                echo "Testing"
                sh '/script/check_linting.sh'
            }
        }
        stage("Deploy"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'git-cred', usernameVariable: 'GITHUB_USERNAME', passwordVariable: 'GITHUB_PASSWORD')]){
                    sh 'echo "$GITHUB_USERNAME $GITHUB_PASSWORD $GIT_LOCAL_BRANCH"'               
                    sh 'bash /script/deploy.sh $GITHUB_USERNAME $GITHUB_PASSWORD'
                }
            }
        }
    }
    post{
        always{
            echo 'Docker stop application'
            sh 'docker stop flask-practice'
            notificationSend()
        }
    }
}

def notificationSend(){
    emailext (
        attachLog: true,
        subject: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
        body: """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
            <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
        recipientProviders: [developers(), buildUser()],
    )
}