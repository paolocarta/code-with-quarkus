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
                    sh "ls -l /tmp/workspace"
                    sh "echo $HOME"
                    sh "ls -la $HOME/.m2"
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
                    sh "ls -l /tmp/workspace/code-with-quarkus/target"

                    sh "buildah --storage-driver=vfs bud --format=oci \
                            --tls-verify=true --no-cache \
                            -f ./src/main/docker/Dockerfile.jvm \
                            -t image-registry.openshift-image-registry.svc:5000/cicd/code-with-quarkus ."

                }            
            }
        }

        stage('Image Push') {

            agent { label 'buildah-x86' }

            options {
                skipDefaultCheckout true
            }

            steps {
                container('buildah') {
                    sh "pwd"
                    sh "id"
                    
                    sh "buildah --storage-driver=vfs push --tls-verify=false \
                            -u jenkins -p eyJhbGciOiJSUzI1NiIsImtpZCI6IjRabE1udXFaU01BY29sNjdfcXVJRVNlQkpXNGJUTWlWZkMzR2lVOF9IVk0ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJjaWNkIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImplbmtpbnMtdG9rZW4tZ3NicGQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiamVua2lucyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImE2OTEyY2Y1LWVlMmQtNGI2ZC1iMjlkLTViMTJlMGM0N2YwMiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpjaWNkOmplbmtpbnMifQ.PAK9f251i9JrTHQJQ9OAHNrtUSnA3Ws9Bq_iB3O3UimZRqFxkiFprnKxcdcYS9mLDaLGJ1NGOm5C_Mjz10ZoHDjo4wcgP7En_p6PjTSuct8vb2LWYSaaGVopJrCt27vCj2VBau7g0FkdDUjqf9xYy_icy0JVQQ-FKYpDUFMEOMmFThAE6J6WLZklHRuGZ3Rp90OlDPPtPD465R22tvurF_Udy1Ue9T1kglY0eNF-4h-2rMoy7DIe7ft3GZxy0cE0m6v9lslojtpHjuqtDbvONRwbUfdyxwvnU0FVnjAsZV7fwS4lUjiwEC1Hltt4lcFXJ6nphfVzAslyWpvmErv05sFPC_1nFO9-Q-awFZakdM0u1F83_PzZgpRrh1TrQCqa-_d6_YGcIdSbGVMiuyVQfh4ER0trxJRH9P8nQVH1Vrphz6WYEdc9qGGQpYukp_Np3Xg8vm4P_ftYN-7ose85Tmp8xWDcGD68rIYKMNI1-ahDMve1bkaUXtD90tdnza4QcA9Bhx3lLn_9xdtoyuRP4-dSQOcrUKsA_2X58s8OY_o2tK2aJAaAya3vVP8CxxfsSea4BQ781Uu9zsm2Gz68quCip0qpoXvBXncHZYQHBLvtQ2XxOzE36oomOZRCiIOXHU8RbrMEcCx2ioNaRi8Ae-hSWFfzRXOSCbNopbf4k5c \
                            image-registry.openshift-image-registry.svc:5000/cicd/code-with-quarkus \
                            docker://image-registry.openshift-image-registry.svc:5000/cicd/code-with-quarkus:latest"

                }            
            }
        }

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
            sh "pwd"
            sh "id"
            sh "hostname"
            sh "echo post-step"
        }
        
        // stage executed after every other post condition has been evaluated, regardless of the Pipeline or stage’s status.
        
    }

}