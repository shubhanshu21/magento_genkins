pipeline {
    agent any

    options {
        copyArtifactPermission('m2-demo-store/DeployDemoStore')
    }

    parameters {
        choice(
            name: 'reference_type',
            choices: [
                'tag',
                'branch'
            ]
        )
        string(
            name: 'reference',
            defaultValue: '2.4.7'
        )
    }

    stages {
        stage('Build') {
            agent any
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: "magento-repo",
                            usernameVariable: "e5669a15f3ce3f484175772baef146d1",
                            passwordVariable: "da292acbb76e8505c95dc193d37f397d"
                        )
                    ]) {
                        sh """
                            composer global config -a http-basic.repo.magento.com "$username" "$password"
                        """
                    }
                }

                script {
                    if ( "${env.reference_type}" == 'branch' ) {
                        env.build_from = "refs/heads/${env.reference}"
                    }

                    else if ( "${env.reference_type}" == 'tag' ) {
                        env.build_from = "refs/tags/${env.reference}"
                    }
                }

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "${env.build_from}"]],
                    userRemoteConfigs: [[name: 'source', url: "https://github.com/magento/magento2.git"]]
                ])

                sh 'rm -rf generated/code generated/metadata pub/static/frontend pub/static/adminhtml'
                sh 'composer install --optimize-autoloader --no-interaction'
                sh 'php bin/magento module:enable --all'
                sh 'php -r \'$config = include("app/etc/config.php"); $stores = include("/var/store-config.php"); file_put_contents("app/etc/config.php", "<?php\\nreturn\\n" . var_export(array_merge($config, $stores), true) . ";");\''
                sh 'php bin/magento setup:di:compile'
                sh 'php bin/magento setup:static-content:deploy -f'
                sh 'rm -rf .git/'
                sh 'mkdir build'
                sh 'tar --exclude=build/output.tar.gz -zcf build/output.tar.gz .'

                archiveArtifacts artifacts: 'build/output.tar.gz', followSymlinks: false, onlyIfSuccessful: true
            }
        }
    }
}
