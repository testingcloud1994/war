pipeline {
    agent any
    environment{
        DOCKERCRED=credentials('docker_registry') 
        Docker_Space='myphpproject'
    }
        
    stages{
        stage("Git Checkout"){
            steps{
                deleteDir();
                sh 'git clone https://github.com/testingcloud1994/war.git'
                }}
        stage("Maven package"){
            steps{
                sh "ls; mvn -nsu -f  war package" 
                echo "scan done "
            }
            post{
                success{
                    archiveArtifacts artifacts: '**/war/target/*.war', fingerprint: true
                }
                failure{
                    echo "project failed"
                }
            }}
        stage("docker image creation"){
            steps{
                sh 'docker build -t suraj_$BUILD_NUMBER:$BUILD_NUMBER war;'
                echo "Image Creation Succeded"
            }}
        stage("login docker"){
            steps{
                sh 'echo $DOCKERCRED_PSW | docker login -u $DOCKERCRED_USR --password-stdin'
                echo "Login Succeded"
            }}
        stage("docker push to image"){
            steps{
                sh 'docker tag suraj_$BUILD_NUMBER:$BUILD_NUMBER $DOCKERCRED_USR/$Docker_Space:$BUILD_NUMBER;docker push $DOCKERCRED_USR/$Docker_Space:$BUILD_NUMBER;'
                echo "Image Pushed Succeded"
            }}      

        stage("cleasning local images and files "){
            steps{  
            sh 'docker rmi suraj_$BUILD_NUMBER:$BUILD_NUMBER $DOCKERCRED_USR/$Docker_Space:$BUILD_NUMBER;'
            deleteDir();
            echo "cleaning completed"
            }}
        }}
