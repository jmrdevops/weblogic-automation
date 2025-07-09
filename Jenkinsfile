pipeline {
    agent any

    environment {
        DEST_HOST = '192.168.3.46'         // 🖥️ Destination host
        SSH_USER  = 'oracle'               // SSH user
        SSH_CRED  = 'jenkins-ssh-key'      // Jenkins credentials ID
        JAR_PATH  = '/tmp/fmw_14.1.1.0.0_wls.jar'  // JAR already on target
    }

    stages {

        stage('Checkout WebLogic Scripts from GitHub') {
            steps {
                git url: 'https://github.com/jmrdevops/weblogic-automation.git', branch: 'main'
            }
        }

        stage('Transfer WebLogic Installer Files') {
            steps {
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        scp oraInst.loc install.rsp create_domain.py ${SSH_USER}@${DEST_HOST}:/tmp/
                    """
                }
            }
        }

        stage('Verify Installer Exists on Target') {
            steps {
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        ssh ${SSH_USER}@${DEST_HOST} '
                            [ -f ${JAR_PATH} ] && echo "✅ Installer exists." || (echo "❌ fmw jar missing at ${JAR_PATH}" && exit 1)
                        '
                    """
                }
            }
        }

        stage('Run WebLogic Installer') {
            steps {
                sshagent (credentials: [env.SSH_CRED]) {
                    sh """
                        ssh ${SSH_USER}@${DEST_HOST} '
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
