pipeline {
    agent any

    environment {
        DEST_HOST = '192.168.3.46'        // 🔁 Updated destination IP
        SSH_USER  = 'oracle'              // 🧑 User with key-based SSH access
        SSH_CRED  = 'jenkins-ssh-key'     // 🔐 Jenkins credentials ID
    }

    stages {
        stage('Checkout GitHub Repo') {
            steps {
                git url: 'https://github.com/jmrdevops/weblogic-automation.git', branch: 'main'
            }
        }

        stage('Transfer WebLogic Install Files') {
            steps {
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        scp oraInst.loc install.rsp ${SSH_USER}@${DEST_HOST}:/tmp/
                    """
                }
            }
        }

        stage('Verify WebLogic Installer') {
            steps {
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        ssh ${SSH_USER}@${DEST_HOST} '
                            [ -f /tmp/fmw.jar ] && echo "✅ fmw.jar found." || (echo "❌ fmw.jar not found in /tmp!" && exit 1)
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
                            java -jar /tmp/fmw.jar -silent -responseFile /tmp/install.rsp -invPtrLoc /tmp/oraInst.loc -ignoreSysPrereqs
                        '
                    """
                }
            }
        }

        stage('Transfer Domain Creation Script') {
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
