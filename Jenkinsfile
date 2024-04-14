pipeline {
    agent any
    environment{
        DOCKERCRED=credentials('docker_registry') 
        Docker_Space='myphpproject'
    }
        
    stages{
        stage("Git Checkout"){
            steps{
                sh 'if ([ -d samplewarfile ]);then rm -r *; fi'
                sh 'git clone https://github.com/testingcloud1994/samplewarfile.git'
                }}
        stage("Maven vulnerability of dependency check "){
            steps{
                sh "mvn -nsu -f  /var/lib/jenkins/workspace/test/samplewarfile/ verify" 
                echo "scan done "
            }}        
        stage(" Sonar code smeill  check(whitebox testing) "){
            steps{
                sh 'ls samplewarfile'
                sh "mvn -nsu -f  /var/lib/jenkins/workspace/test/samplewarfile/  sonar:sonar -Dsonar.projectKey=myfirst_project -Dsonar.host.url=http://localhost:9000 -Dsonar.login=7747963ab521bc6293c15e2c94e8d0baaa07485b"
                echo "scan done "
            }
            post{
                success{
                    archiveArtifacts artifacts: '**/samplewarfile/target/*.war', fingerprint: true
                    archiveArtifacts artifacts: '**/samplewarfile/target/*.html', fingerprint: true
                    /*archiveArtifacts artifacts: , fingerprint: true   */
                }
                failure{
                    echo "project failed"
                }
            }}
        stage("docker image creation"){
            steps{
                sh 'docker build -t suraj_$BUILD_NUMBER:$BUILD_NUMBER samplewarfile;'
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
