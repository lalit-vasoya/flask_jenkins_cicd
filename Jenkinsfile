pipeline {
    agent { label 'slave1'}
    environment {
        JENKINS_GIT_LTS_BRANCH = 'feature/master_slave'
    }
    stages {
        stage("BUILD") {
            steps {
                sh 'docker-compose up --build -d '
            }
        }
        stage("CHECK-linting") {
            steps {
                echo "Linting"
                sh 'docker exec -i flask-practice bash -c "flake8"'
            }
        }
        stage("TEST"){
            steps {
                echo "Testing"
                sh 'docker exec -i flask-practice bash -c "pytest"'
            }
        }
        stage("Deploy"){
            agent { label 'flask-practice'}
            when {
                expression { sha1 == 'develop'}
            }
            steps{
                echo "Deploying the ${sha1} branch on server..."
                sh 'docker-compose up --build -d'
            }
        }
    }
    post{
        always{
            echo 'Docker stop application....!'
            sh 'docker stop flask-practice'
            notificationSend()
            googleChatNotifation()
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
        to: "${env.ghprbActualCommitAuthorEmail}"
    )
}

def googleChatNotifation(){
    googlechatnotification(
        url: "id:google-chat",
        message: "${env.JOB_NAME} Done ${env.JOB_NAME}: ${env.BUILD_NUMBER} url: ${env.BUILD_URL}",
        sameThreadNotification: true
    )
}
