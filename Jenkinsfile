pipeline {
    agent any

    environment {
        DEST_HOST = '192.168.1.250'
        SSH_USER  = 'oracle'
        SSH_CRED  = 'jenkins-ssh-key'
    }

    stages {
        stage('Checkout WebLogic Automation Scripts from GitHub') {
            steps {
                git url: 'https://github.com/your-org/weblogic-automation.git', branch: 'main'
            }
        }

        stage('Copy WebLogic Install Files to Destination') {
            steps {
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        scp oraInst.loc ${SSH_USER}@${DEST_HOST}:/tmp/
                        scp install.rsp ${SSH_USER}@${DEST_HOST}:/tmp/
                    """
                }
            }
        }

        stage('Verify Installer Exists on Destination') {
            steps {
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        ssh ${SSH_USER}@${DEST_HOST} '
                            [ -f /tmp/fmw_14.1.1.0.0_wls.jar ] && echo "fmw_14.1.1.0.0_wls.jar exists" || (echo "❌ fmw.jar not found in /tmp" && exit 1)
                        '
                    """
                }
            }
        }

        stage('Install WebLogic') {
            steps {
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        ssh ${SSH_USER}@${DEST_HOST} '
                            java -jar /tmp/fmw_14.1.1.0.0_wls.jar -silent -responseFile /tmp/install.rsp -invPtrLoc /tmp/oraInst.loc -ignoreSysPrereqs
                        '
                    """
                }
            }
        }

        stage('Copy Domain Creation Script') {
            steps {
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        scp create_domain.py ${SSH_USER}@${DEST_HOST}:/tmp/
                    """
                }
            }
        }

        stage('Create WebLogic Domain') {
            steps {
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        ssh ${SSH_USER}@${DEST_HOST} '
                            /u01/app/oracle/middleware/oracle_common/common/bin/wlst.sh /tmp/create_domain.py
                        '
                    """
                }
            }
        }
    }
}
