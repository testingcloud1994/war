pipeline {
    agent {
        docker { image 'node:20.11.1-alpine3.19' }
    }
    environment{
        DOCKERCRED=credentials('docker_registry') 
        Docker_Space='myphpproject'
        git_branch='test_tools'
    }
        
    stages{
        stage("prepare node")
        {steps{
          sh "sudo apt-get update"
          sh " sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin "
          sh " sudo apt-get install git maven"

        }}
        stage("Git Checkout"){
            steps{
                deleteDir();
                sh 'git clone -b $git_branch  https://github.com/testingcloud1994/war.git'
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

        
        stage("deploy to test env"){
          agent any
             steps{ 
               /* script {
                    env.RELEASE_SCOPE = input message: 'User input required', ok: 'Release!',
                            parameters: [choice(name: 'RELEASE_SCOPE', choices: 'patch\nminor\nmajor', description: 'What is the release scope?')]
                }*/
                echo "${env.RELEASE_SCOPE}"
                dir("war"){
                    dir("yaml"){
                        sh ""
                        sh """
                        export KUBECONFIG=/home/suraj/.kube/config ;
                        /usr/local/bin/kubectl create  ns testing
                        cat <<EOF | /usr/local/bin/kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: testing
  name: mytomcatwar
  labels:
    app: mybestapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mybestapp
  template:
    metadata:
      labels:
        app: mybestapp
    spec:
      containers:
      - name: nginx
        image: testingcloud1994/myphpproject:$BUILD_NUMBER
        ports:
        - containerPort: 8080

---

apiVersion: v1
kind: Service
metadata:
  namespace: testing
  name: my-service
spec:
  type: NodePort
  selector:
    app: mybestapp
  ports:
    - protocol: TCP
      port: 8888
      targetPort: 8080
      nodePort: 30007

EOF
                        
                        
                        """
                        sleep(time:150 ,unit:"SECONDS") 
                        sh """
                        export KUBECONFIG=/home/suraj/.kube/config 
                        /usr/local/bin/kubectl delete  ns testing
                        """
                }}
                //sh "kubectl get ns"
                // create deployement and push updated yml to git 
                // deplo image to kubernetes
                //set nodeto allow localhost  url 
                // sleep for 4 min to test 
                 //clean everything to back nrmal 
             }}
        stage("cleasning local images and files "){
            steps{  
            sh 'docker rmi suraj_$BUILD_NUMBER:$BUILD_NUMBER $DOCKERCRED_USR/$Docker_Space:$BUILD_NUMBER;'
            deleteDir();
            echo "cleaning completed"
            }}
        }}