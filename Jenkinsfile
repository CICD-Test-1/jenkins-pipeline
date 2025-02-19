stage('QA Testing & Report') {
            steps {
                script {
                        def gitBranch = "main" 

                    withCredentials([
                        usernamePassword(credentialsId: 'windows-cred', usernameVariable: 'seleniumhostUser', passwordVariable: 'seleniumhostPassword'),
                        usernamePassword(credentialsId: 'git-cred', usernameVariable: 'gitUser', passwordVariable: 'gitPassword')
                        usernamePassword(credentialsId: 'ext-cred', usernameVariable: 'seleniumhost', passwordVariable: 'gitRepoURL')
                    ]) {

                        def remote = [:]
                        remote.name = 'windowssh'
                        remote.host = seleniumhost
                        remote.user = seleniumhostUser
                        remote.password = seleniumhostPassword
                        remote.allowAnyHosts = true
                        
                        // Check if repository already exists; if so, delete it and clone again
                        sshCommand remote: remote, command: "if exist MD_Automation (rmdir /s /q MD_Automation)"
                        sshCommand remote: remote, command: "git clone --branch ${gitBranch} https://${gitUser}:${gitPassword}@${gitRepoURL} MD_Automation"
                        
                        sshCommand remote: remote, command: "pip install -r MD_Automation/requirements.txt"
                        sshCommand remote: remote, command: "pytest -v -s -m aws MD_Automation\\tests\\test_create_vpc.py::TestCreateVpc::test_network_create"
                        
                        sh """
                        #!/bin/bash
                        apt-get install -y sshpass zip
                        sshpass -p ${seleniumhostPassword} scp -o StrictHostKeyChecking=no ${seleniumhostUser}@${seleniumhost}:C://Users//${seleniumhostUser}//report.html .
                        sshpass -p ${seleniumhostPassword} scp -o StrictHostKeyChecking=no ${seleniumhostUser}@${seleniumhost}:C://Users//${seleniumhostUser}//assets//style.css .
                        mkdir -p assets && mv style.css assets
                        zip -r test-report.zip report.html assets
                        """
                        archiveArtifacts artifacts: 'test-report.zip', allowEmptyArchive: true
                    }
                }
            }
        }
