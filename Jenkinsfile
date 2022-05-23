pipeline {
    agent any
    parameters{
        booleanParam(name: 'PROMOTE', defaultValue: false, description: 'should publish')
    }
    stages {
        stage('Build_image') {
            steps {
                script {    
                    env.GIT_COMMIT_REV = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
                }
                sh "docker build --file Dockerfile-b --tag appbuildimage:latest ."
                sh "docker volume create invol"
                sh "docker volume create outvol"
            }     
        }
        stage('Copy_Build') {
            agent {
                docker {
                    image'appbuildimage:latest'
                    args '-v invol:/input  -v outvol:/output  --user root'
                    reuseNode true
                    }
                }
            steps {
                sh 'rm -rf /input/*'
                sh 'rm -rf /output/*'
                sh 'cp -r /calculator/*  /input/'
                sh 'cp -r  /calculator/src /output/'   
                sh 'cp -r  /calculator/test /output/'
                sh 'cp -r  /calculator/CMakeLists.txt /output/'
                sh 'cp -r  /calculator/bin /output/'
                sh 'cp -r  /calculator/CMakeFiles /output/'
                sh 'cp -r  /calculator/Makefile /output/'
                sh 'cp -r  /calculator/CTestTestfile.cmake /output/'
                sh 'cp -r  /calculator/lib /output/'
               // sh 'cp -r  /calculator/instruction.txt /output/'
            }
        }
        stage('Build_Test') {
        
            steps {
                sh "docker build --file Dockerfile-t --tag apptestimage:latest ."
            }
        }
        stage('Test') {
             agent {
                docker {
                    image'apptestimage:latest'
                    args '-v invol:/input  -v outvol:/output  --user root'
                    reuseNode true
                    }
                }
            steps {
                sh 'cd /output && make test' 
     
            }
        }
        stage('Deploy') {
             agent {
                docker {
                    image'apptestimage:latest'
                    args '-v outvol:/output  --user root'
                    reuseNode true
                    }
                }
            steps {
                sh 'cd /output/bin &&  ./calculator.x' 
            }
        }
         stage('Prepublish') {
             agent {
                docker {
                    image'apptestimage:latest'
                    args '-v invol:/input  -v outvol:/output  --user root'
                    reuseNode true
                    }
                }
            steps {
                sh 'rm -rf publish_folder'
                sh 'rm -rf checksum.txt'
                sh 'mkdir  publish_folder'
                sh 'rm -f publish_folder*.tar.gz'
                sh 'cp -r /output/src ./publish_folder/' 
                sh 'cp /output/build_cmake/CMakeLists.txt ./publish_folder/'
                sh 'cp /output/Makefile ./publish_folder'
            }
        }
         stage('Publish') {
             when{
                 environment name: 'PROMOTE', value: 'true'
             }
            steps {

                sh 'tar -zcvf publish_folder${GIT_COMMIT_REV}.tar.gz ./publish_folder'
                sh 'cat publish_folder${GIT_COMMIT_REV}.tar.gz | sha512sum > checksum.txt'
                archiveArtifacts artifacts: 'publish_folder*.tar.gz', fingerprint: true   
                archiveArtifacts artifacts: 'checksum.txt', fingerprint: true      
            }
        }
    }
    post{
        always{
           
            sh 'docker rmi appbuildimage'   
            sh 'docker rmi apptestimage'
        }
    }
}
