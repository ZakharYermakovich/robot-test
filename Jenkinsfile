pipeline {
    agent {
        node {
            label 'java14'
        }
    }
    options {
        timestamps()
        buildDiscarder(logRotator(daysToKeepStr: '21', numToKeepStr: '50'))
        disableConcurrentBuilds()
        /* office365ConnectorWebhooks([[
                                            startNotification    : false,
                                            notifyAborted        : true,
                                            notifyFailure        : true,
                                            notifyNotBuilt       : false,
                                            notifySuccess        : true,
                                            notifyUnstable       : true,
                                            notifyRepeatedFailure: true,
                                            notifyBackToNormal   : true,

                                            name                 : "Jenkins Notifications",
                                            url                  : 'https://allot365.webhook.office.com/webhookb2/fd7d1a87-2075-4e43-b4f0-f18bfba1f435@789e5ff8-0396-414e-803b-13a424e9f5d2/JenkinsCI/8d9b9f2beae141a2a0c2927e51470839/5acaaf99-7ad6-4f57-b9c3-cf407860e5e2'
                                    ]]) */
    }
    parameters {
        gitParameter name: 'BRANCH_NAME', branchFilter: 'origin/(.*)', defaultValue: 'develop', selectedValue: 'DEFAULT', sortMode: 'ASCENDING', type: 'PT_BRANCH'

        choice(name: 'COMPONENT_TO_TEST', choices: ['all', 'nx', 'smp', 'aos', 'tgm'], description: 'Choose a component to test')
        choice(name: 'FEATURE_TO_TEST', choices: ['all', 'hhe', 'api', 'get', 'some_feature'], description: 'Choose a feature to test')
        choice(name: 'VERSION_TO_TEST', choices: ['all', '17.3.10', '17.4.10', '16.3.10'], description: 'Choose product version')

        string(name: 'NX_IP', description: 'Specify NX IP')
        string(name: 'NE_IP', description: 'Specify NE IP')
        string(name: 'SMP_IP', description: 'Specify SMP IP')
    }

    stages {
        stage("Build init") {
            steps {
                script {
                    def BUILD_TRIGGER_BY = "${currentBuild.getBuildCauses()[0].shortDescription} [${currentBuild.getBuildCauses()[0].userId}]".replace("Started by user ", "")
                    currentBuild.displayName = "#${BUILD_NUMBER}-${BRANCH_NAME}"
                    currentBuild.description =
                            """
	                Triggered by: ${BUILD_TRIGGER_BY}
	                <br>Triggered from: ${BUILD_URL}
	                <br>
	                <br>Branch: ${BRANCH_NAME}
	                """
                }
            }
        }
        stage("Download sources") {
            steps {
                git branch: "${params.BRANCH_NAME}", credentialsId: 'svc_bitbucket', url: 'https://bitbucket.rdlab.local/scm/aut/stf-robot.git'
            }
        }

        stage('Install python and robot framework') {
            steps {
                echo 'Upgrade yum - sudo yum -y update'
                sh 'sudo yum -y update'

                echo 'yum install epel-release -y'
                sh 'yum install epel-release -y'
                echo 'yum install dnf -y'
                sh 'yum install dnf -y'

                echo 'Install python'
                sh 'sudo dnf install python3'
                sh 'python3 -V'

                echo 'Upgrade pip3'
                sh 'sudo -H pip3 install --upgrade pip'

                echo 'install robotframework'
                sh 'pip3 install robotframework'

                echo 'install robotframework-sshlibrary'
                sh 'pip3 install robotframework-sshlibrary'

                echo 'install json library'
                sh 'pip3 install robotframework-jsonlibrary'

                echo 'install requests library'
                sh 'pip3 install robotframework-requests'
            }
        }
        stage('Run tests') {
            steps {
                script {
                    echo 'Run tests'
                    String cmd = "robot "
                    if (env.NX_IP) {
                        cmd += "-v NX_IP:$NX_IP "
                    }
                    if (env.NE_IP) {
                        cmd += "-v NE_IP:$NE_IP "
                    }
                    if (env.SMP_IP) {
                        cmd += "-v SMP_IP:$SMP_IP "
                    }
                    if (!params.COMPONENT_TO_TEST.equals('all')){
                        cmd += "--suite *${COMPONENT_TO_TEST}* "
                    }
                    if (!params.FEATURE_TO_TEST.equals('all')){
                        cmd += "--test *${FEATURE_TO_TEST}* "
                    }
                    if (!params.VERSION_TO_TEST.equals('all')){
                        cmd += "--include ${VERSION_TO_TEST} "
                    }
                    //cmd +='.'
                    cmd += "test/suites/components/nx/nx-base-test.robot"
                    // cmd == "robot --suite *${COMPONENT_TO_TEST}* --test *${FEATURE_TO_TEST}* --include ${VERSION_TO_TEST} ."
                    echo "${cmd}"
                    sh "${cmd}"

                }
            }
        }
    }

    post {
        always {
            archive(includes: '*.html')
            echo 'Result: ' + currentBuild.currentResult

            echo "Cleanup workspace folder: ${env.WORKSPACE}"
            dir("${env.WORKSPACE}") {
                deleteDir()
            }
        }
        success {
            echo 'SUCCESS!'
            script { RUN_RESULT = "SUCCESS" }
        }
        unstable {
            echo 'UNSTABLE'
            script { RUN_RESULT = "UNSTABLE" }
        }
        failure {
            echo 'FAILURE'
            script { RUN_RESULT = "FAILURE" }
        }
    }
}
