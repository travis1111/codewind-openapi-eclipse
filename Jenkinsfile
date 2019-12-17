#!groovy​

pipeline {
    agent {
        label "docker-build"
    }
    
    tools {
        jdk 'oracle-jdk8-latest'
    }
    
    options {
        timestamps() 
        skipStagesAfterUnstable()
    }
    
    stages {

        stage('Build') {
            steps {
                script {
                    println("Starting codewind-openapi-eclipse build ...")
                        
                    def sys_info = sh(script: "uname -a", returnStdout: true).trim()
                    println("System information: ${sys_info}")
                    println("JAVE_HOME: ${JAVA_HOME}")
                    
                    sh '''
                        java -version
                        which java    
                    '''
                    
                    dir('dev') { sh './gradlew --stacktrace' }
                }
            }
        } 
        stage('Test') {
            steps {
                sh '''#!/usr/bin/env bash
                    docker build --no-cache -t test-image ./dev
                    docker run -v /var/run/docker.sock:/var/run/docker.sock -v ./dev:/development test-image
                '''
            }
        }  
        stage('Deploy') {
            steps {
                sshagent ( ['projects-storage.eclipse.org-bot-ssh']) {
                    println("Deploying codewind-openapi-eclipse to downoad area...")
                  
                    sh '''
                        export REPO_NAME="codewind-openapi-eclipse"
                        export OUTPUT_NAME="codewind-openapi-eclipse"
                        export OUTPUT_DIR="$WORKSPACE/dev/ant_build/artifacts"
                        export DOWNLOAD_AREA_URL="https://download.eclipse.org/codewind/$REPO_NAME"
                        export LATEST_DIR="latest"
                        export BUILD_INFO="build_info.properties"
                        export sshHost="genie.codewind@projects-storage.eclipse.org"
                        export deployDir="/home/data/httpd/download.eclipse.org/codewind/$REPO_NAME"
                    
                        if [ -z $CHANGE_ID ]; then
                            UPLOAD_DIR="$GIT_BRANCH/$BUILD_ID"
                            BUILD_URL="$DOWNLOAD_AREA_URL/$UPLOAD_DIR"
                  
                            ssh $sshHost rm -rf $deployDir/$GIT_BRANCH/$LATEST_DIR
                            ssh $sshHost mkdir -p $deployDir/$GIT_BRANCH/$LATEST_DIR
                            cp $OUTPUT_DIR/$REPO_NAME-*.zip $OUTPUT_DIR/$REPO_NAME.zip
                            scp $OUTPUT_DIR/$REPO_NAME.zip $sshHost:$deployDir/$GIT_BRANCH/$LATEST_DIR/$REPO_NAME.zip
                        
                            echo "# Build date: $(date +%F-%T)" >> $OUTPUT_DIR/$BUILD_INFO
                            echo "build_info.url=$BUILD_URL" >> $OUTPUT_DIR/$BUILD_INFO
                            SHA1=$(sha1sum ${OUTPUT_DIR}/${OUTPUT_NAME}.zip | cut -d ' ' -f 1)
                            echo "build_info.SHA-1=${SHA1}" >> $OUTPUT_DIR/$BUILD_INFO
                  
                            unzip $OUTPUT_DIR/$REPO_NAME-*.zip -d $OUTPUT_DIR/repository
                            scp -r $OUTPUT_DIR/repository $sshHost:$deployDir/$GIT_BRANCH/$LATEST_DIR/repository
                            scp $OUTPUT_DIR/$BUILD_INFO $sshHost:$deployDir/$GIT_BRANCH/$LATEST_DIR/$BUILD_INFO
                            rm $OUTPUT_DIR/$BUILD_INFO
                            rm $OUTPUT_DIR/$REPO_NAME.zip
                            rm -rf $OUTPUT_DIR/repository
                        else
                            UPLOAD_DIR="pr/$CHANGE_ID/$BUILD_ID"
                        fi
                        
                        ssh $sshHost rm -rf $deployDir/${UPLOAD_DIR}
                        ssh $sshHost mkdir -p $deployDir/${UPLOAD_DIR}
                        scp -r $OUTPUT_DIR/* $sshHost:$deployDir/${UPLOAD_DIR}
                    '''
                }
            }
        }    
           
    }   
    post {
      always {
        junit './dev/results.xml'
      }
   }   
}

