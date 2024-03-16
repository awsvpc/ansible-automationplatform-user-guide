<pre>
Add the SSH keys to Jenkins Credentials and call them from the pipeline. You do need the ansible plugin to use this pipeline.

pipeline {
    agent { label 'aws' }
    environment {
        SSHKEY = credentials('jenkins-ansible')
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        ANSIBLE_SCP_IF_SSH = 'True'
    }
    stages {
        stage('dev') {
            steps {
                ansiblePlaybook colorized: true, credentialsId: 'jenkins-ansible-key', disableHostKeyChecking: true, installation: 'ansible-2.6.3', inventory: 'inventory/development.yaml', playbook: 'updates.yaml'
            }
        }
        stage('prod') {
            when { branch 'master' }
            steps {
                ansiblePlaybook colorized: true, credentialsId: 'jenkins-ansible-key', disableHostKeyChecking: true, installation: 'ansible-2.6.3', inventory: 'inventory/production.yaml', playbook: 'updates.yaml'
            }
        }
    }
    post {
        always {
            sendNotifications(currentBuild.currentResult)
        }
    }
}
</pre>

