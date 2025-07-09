pipeline {
    agent any

    environment {
        DEST_HOST = '192.168.3.46'                  // Destination WebLogic host
        SSH_USER  = 'oracle'                        // Remote SSH user
        SSH_CRED  = 'jenkins-ssh-key'               // Jenkins credential ID (SSH private key)
        JAR_PATH  = '/tmp/fmw_14.1.1.0.0_wls.jar'    // Already copied manually
    }

    stages {

        stage('Checkout Scripts from GitHub') {
            steps {
                echo "🔄 Checking out WebLogic install scripts..."
                git url: 'https://github.com/jmrdevops/weblogic-automation.git', branch: 'main'
            }
        }

        stage('Transfer WebLogic Response Files') {
            steps {
                echo "🚀 Transferring oraInst.loc, install.rsp, and create_domain.py to ${DEST_HOST}..."
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        set -e
                        scp -o StrictHostKeyChecking=no oraInst.loc install.rsp create_domain.py ${SSH_USER}@${DEST_HOST}:/tmp/
                    """
                }
            }
        }

        stage('Verify Installer Exists on Target') {
            steps {
                echo "🔍 Verifying WebLogic installer JAR exists at ${JAR_PATH}..."
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_USER}@${DEST_HOST} '
                            if [ -f "${JAR_PATH}" ]; then
                                echo "✅ Installer found: ${JAR_PATH}";
                            else
                                echo "❌ ERROR: Installer not found at ${JAR_PATH}";
                                exit 1;
                            fi
                        '
                    """
                }
            }
        }

        stage('Run WebLogic Installer') {
            steps {
                echo "🛠️ Running WebLogic installer on ${DEST_HOST}..."
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_USER}@${DEST_HOST} '
                            set -e
                            java -jar ${JAR_PATH} \\
                                -silent \\
                                -responseFile /tmp/install.rsp \\
                                -invPtrLoc /tmp/oraInst.loc \\
                                -ignoreSysPrereqs \\
                                -novalidation
                        '
                    """
                }
            }
        }

        stage('Create WebLogic Domain') {
            steps {
                echo "🏗️ Creating WebLogic domain using WLST..."
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_USER}@${DEST_HOST} '
                            /u01/app/oracle/middleware/oracle_common/common/bin/wlst.sh /tmp/create_domain.py
                        '
                    """
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Pipeline failed! Please check the logs above for details.'
        }
        success {
            echo '✅ WebLogic installation and domain creation completed successfully.'
        }
    }
}
