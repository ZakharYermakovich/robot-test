pipeline {
 
    options {
        timestamps()
        buildDiscarder(logRotator(daysToKeepStr: '21', numToKeepStr: '50'))
        disableConcurrentBuilds()
    }
	
    parameters {
        gitParameter name: 'BRANCH_NAME', branchFilter: 'origin/(.*)', defaultValue: 'main', selectedValue: 'DEFAULT', sortMode: 'ASCENDING', type: 'PT_BRANCH'    
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
                    cmd += "test/suites/components/testrail"
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
