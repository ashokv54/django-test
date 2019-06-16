#!groovy

node {

    try {
        
        stage 'Checkout'
            echo 'Git Checkout'
            checkout scm
            

            sh 'git log HEAD^..HEAD --pretty="%h %an - %s" > GIT_CHANGES'
            def lastChanges = readFile('GIT_CHANGES')
            slackSend color: "warning", message: "Started `${env.JOB_NAME}#${env.BUILD_NUMBER}`\n\n_The changes:_\n${lastChanges}"

        stage 'Test'
            echo 'Tesitng...'
            sh 'virtualenv env -p python3.6'
            sh '. env/bin/activate'
            sh 'env/bin/pip install -r requirements.txt'
            sh 'env/bin/python3.6 manage.py test'
            junit 'reports/junit.xml'
        stage ("Run Unit/Integration Tests") {
    def testsError = null
    try {
        sh '''
            source ../bin/activate
            python <relative path to manage.py> jenkins
            deactivate
           '''
    }
    catch(err) {
        testsError = err
        currentBuild.result = 'FAILURE'
    }
    finally {
        junit 'reports/junit.xml'

        if (testsError) {
            throw testsError
        }
    }

}


        stage 'Deploy'
            echo 'Deploying'
            sh 'whoami'

        stage 'Publish results'
            slackSend color: "good", message: "Build successful: `${env.JOB_NAME}#${env.BUILD_NUMBER}` <${env.BUILD_URL}|Open in Jenkins>"
    }

    catch (err) {
        slackSend color: "danger", message: "Build failed :face_with_head_bandage: \n`${env.JOB_NAME}#${env.BUILD_NUMBER}` <${env.BUILD_URL}|Open in Jenkins>"

        throw err
    }

}

