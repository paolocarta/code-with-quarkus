pipeline {

    agent any

    // agent {
    //     kubernetes {
    //         yamlFile 'pod-template-buildah.yaml'
    //     }
    // }

    parameters {

        booleanParam(defaultValue: false,description: 'Deploy to Prod Environment ?', name: 'DEPLOY_PROD')
        string(defaultValue: "maven-settings-general", description: 'Maven Settings File Id', name: 'MAVEN_SETTINGS_ID')
        string(defaultValue: "http://baifuseqa01.internal.com/nexus", description: 'Nexus URL', name: 'NEXUS_URL')
        string(defaultValue: "artifacts", description: 'Nexus hosted repo name', name: 'NEXUS_HOSTED_NAME_RELEASES')
        string(defaultValue: "artifacts-snapshots", description: 'Nexus hosted snapshots repo name', name: 'NEXUS_HOSTED_NAME_SNAPSHOTS')
        
        string(defaultValue: "artifacts-snapshots", description: 'Nexus hosted snapshots repo name', name: 'NEXUS_HOSTED_NAME_SNAPSHOTS')

        string(defaultValue: 'test@gmail.com', description: 'Notification recipients', name: 'EMAIL_RECIPIENT_LIST')
    }
    
    options {
        
        // use colors in Jenkins console output
        // ansiColor('xterm')
        
        // discard old builds
        buildDiscarder logRotator(daysToKeepStr: '30', numToKeepStr: '45')
        
        // disable concurrent builds and disable build resume
        // disableConcurrentBuilds()
        // disableResume()

        // timeout on whole pipeline job
        timeout(time: 1, unit: 'HOURS')
    }
    
    triggers {
        pollSCM "*/2 * * * *"
    }

    environment {

        PLACEHOLDER_VARIABLE   = 'TBD'
    }


  
stages {

    stage("Initialize") {
        
        agent any

        options {
            skipDefaultCheckout true
        }

        steps {
            script {
                    // notifyBuild('STARTED')
                    echo "Build number: ${BUILD_NUMBER} - Build id: ${env.BUILD_ID} on Jenkins instance: ${env.JENKINS_URL}"

                    echo "Deploy to QA? :: ${params.DEPLOY_QA}"

                    echo "Maven Settings file id? :: ${params.MAVEN_SETTINGS_ID}"
                    echo "NEXUS URL? :: ${params.NEXUS_URL}"
                    sh "printenv | sort"

            }
        }
    }

    stage('Code Build and Test') {

        agent { label 'maven' }

        steps {

            configFileProvider([configFile(fileId: "${params.MAVEN_SETTINGS_ID}", variable: 'MAVEN_SETTINGS')]) {
                
                sh "pwd"
                sh "id"
                sh "ls -l /tmp"
                sh "ls -l /tmp/workspace"
                sh "echo $HOME"
                sh "ls -l $HOME"
                sh "mvn -U -B package -s ${MAVEN_SETTINGS}"
            }
        }
    }

    stage('Image Build') {

        agent { label 'buildah-x86' }

        options {
            skipDefaultCheckout true
        }

        // when {
        //     // fake condition for now
        //     branch 'master' 
        // }

        steps {
            container('buildah') {
                sh "pwd"
                sh "id"
                sh "echo $HOME"
                sh "ls -l"
                sh "ls -l /tmp/workspace/code-with-quarkus/target"
                    
                sh "buildah --storage-driver=vfs bud --format=oci \
                        --tls-verify=true --no-cache \
                        -f ./src/main/docker/Dockerfile.jvm -t clamer/code-with-quarkus ./target"
            }            
        }
    }

    // stage('Push Docker Image') {

    //     agent { label 'buildah-x86' }
        
    //     steps {
    //         container('buildah') {
    //             echo 'Pushing Image'
    //         }            
    //     }
    // }

    // stage('Update GitOps Repo') {
        
    //     // when {
    //     //     branch 'master'
    //     // }
        
    //     steps {
            
    //     }
    // }

  }

    post {

        // stage executed always, regardless of the completion status of the Pipeline’s or stage’s run
        always {
            // cleanWs()
            sh "rm -rf /tmp/workspace"
        }
        
        // stage executed after every other post condition has been evaluated, regardless of the Pipeline or stage’s status.
        
    }

}