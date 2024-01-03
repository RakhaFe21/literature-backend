pipeline {
    agent any
    environment{
        credential = 'github-appserver'
        server = 'team3@103.127.132.63'
        directory = '/home/team3/literature-backend'
        branch = 'main'
        service = 'backend'
        image = 'rakhafe/backend'
    }
    stages {
        stage('Pull code dari repository'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    docker compose down
                    cd ${directory}
                    git pull origin ${branch}
                    exit
                    EOF'''
                }
            }
        }
        stage('Building application'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    cd ${directory}
                    docker compose up -d database
                    docker rmi  ${image}:latest
                    docker build -t ${image}:latest .
                    exit
                    EOF'''
                }
            }
        }
        stage('Testing application'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    cd ${directory}
                    docker run --name test_be -p 5000:5000 -d ${image}:latest
                    curl localhost:5000
                    docker stop test_be
                    docker rm test_be
                    exit
                    EOF'''
                }
            }
        }
        stage('Deploy aplikasi on top docker'){
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF
                    docker compose up -d 
                    cd ${directory}
                    exit
                    EOF'''
                }
            }
        }
        stage('Push image to docker hub'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    cd ${directory}
                    docker push ${image}:latest
                    exit
                    EOF'''
                }
            }
        }
        stage('send notification to discord'){
            steps {
                discordSend description: "Backend Pipeline", footer: "voilaaa!!!", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: "https://discord.com/api/webhooks/1192097833042579536/qInbfRCW9G8UdivrNlhqeO-AvSijRIdWYbTaGKgKx8sSDR8bCHllZrZ8dPQR3HwKKjau"
            }
        }
    }
}