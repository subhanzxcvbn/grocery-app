
@Library('jenkins-shared-library') _

properties([
    parameters([
        string(
            name: 'IMAGE_TAG',
            defaultValue: 'latest',
            description: 'Docker image version to deploy'
        )
    ])
])

node {

    try {

        stage('Code CheckOut') {
            checkoutcode(
                'https://github.com/subhanzxcvbn/grocery-app.git',
                'main'
            )
        }

        stage('Build Frontend') {
            buidImages(
                ("grocery-frontend:${params.IMAGE_TAG}").toString(),
                'client'
            )
        }

        stage('Build Backend') {
            buidImages(
                ("grocery-backend:${params.IMAGE_TAG}").toString(),
                'server'
            )
        }

        stage('Push Frontend to Docker Hub') {
            pushDocker(
                ("grocery-frontend:${params.IMAGE_TAG}").toString(),
                'grocery-frontend',
                'jenkinscred',
                params.IMAGE_TAG
            )
        }

        stage('Push Backend to Docker Hub') {
            pushDocker(
                ("grocery-backend:${params.IMAGE_TAG}").toString(),
                'grocery-backend',
                'jenkinscred',
                params.IMAGE_TAG
            )
        }

        stage('SSH Connection Test') {
            sshagent(['remote-deployment']) {
                sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@13.220.83.125 "hostname"
                '''
            }
        }

        stage('Deploy') {
            withCredentials([
                file(credentialsId: 'backend-env', variable: 'ENV_FILE')
            ]) {
                sshagent(['remote-deployment']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no "$ENV_FILE" ubuntu@13.220.83.125:/tmp/grocery.env

                        ssh -o StrictHostKeyChecking=no ubuntu@13.220.83.125 '
                            mkdir -p ~/apps/grocery-app/server &&
                            install -m 600 /tmp/grocery.env ~/apps/grocery-app/server/.env &&
                            rm -f /tmp/grocery.env
                        '
                    '''
                }

                remoteDeploy(params.IMAGE_TAG)
            }
        }

    }

    catch (err) {

        emailext(
            subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
                Grocery Application Pipeline Failed!

                Job: ${env.JOB_NAME}
                Build: #${env.BUILD_NUMBER}
                Image Tag: ${params.IMAGE_TAG}
                Build URL: ${env.BUILD_URL}
            """,
            to: "maqsoodsubhan01@gmail.com"
        )

        throw err
    }

    finally {

        if (currentBuild.currentResult == 'SUCCESS') {

            emailext(
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    Grocery Application Pipeline Successful!

                    Job: ${env.JOB_NAME}
                    Build: #${env.BUILD_NUMBER}
                    Image Tag: ${params.IMAGE_TAG}
                    Build URL: ${env.BUILD_URL}
                """,
                to: "maqsoodsubhan01@gmail.com"
            )
        }
    }
}
