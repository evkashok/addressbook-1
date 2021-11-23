pipeline{
    agent any
    tools{
        jdk 'myjava'
        maven 'mymaven'
    }
    environment{
        APP_NAME='java-mvn-app'
    }
    parameters{
        choice(name:'VERSION',choices:['1.1.0','1.2.0','1.3.0'],description:'version of the code')
        booleanParam(name: 'executeTests',defaultValue: true,description:'tc validity')
    }
    stages{
        stage("COMPILE"){          
            steps{
                script{
                    echo "Compiling the code"
                    sh 'mvn compile'
                }
            }
        }
        stage("UNITTEST"){           
            when{
                expression{
                    params.executeTests == true
                }
            }
            steps{
                script{
                    echo "Testing the code"
                    sh 'mvn test'
                }
            }
            post{
                always{
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
         stage("PACKAGE"){
                     steps{
                script{
                    echo "Packaging the code"
                    sh 'mvn package'
                }
            }
        }
         stage("BUILD THE DOCKER IMAGE"){       
            steps{
                script{
                    echo "BUILDING THE DOCKER IMAGE"
                   sh 'sudo systemctl start docker -y'
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        
                        sh 'sudo docker build -t devopstrainer/java-mvn-privaterepos:$BUILD_NUMBER .'
                        sh 'sudo docker login -u $USER -p $PASS'
                        sh 'sudo docker push devopstrainer/java-mvn-privaterepos:$BUILD_NUMBER'
                }
            }
        }
         }
        stage("DEPLOYONec2"){
            steps{
                script{
                    echo "Deploying the app"
                    echo "Deploying version ${params.VERSION}"
                   sshagent(['deploy-server-key']) {
                       withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
            sh "ssh -o StrictHostKeyChecking=no ec2-user@13.233.87.48 'sudo amazon-linux-extras install docker -y'"
            sh "ssh -o StrictHostKeyChecking=no ec2-user@13.233.87.48 'sudo systemctl start docker -y'"
            sh "ssh -o StrictHostKeyChecking=no ec2-user@13.233.87.48 'sudo docker login -u $USER -p $PASS'"
            sh "ssh -o StrictHostKeyChecking=no ec2-user@13.233.87.48 'sudo docker run -itd -P devopstrainer/java-mvn-privaterepos:$BUILD_NUMBER'"
}
                }
            }
    }
}
}
}