def config

pipeline {
    agent any

    stages {
        stage('Read Pipeline Config') {
            steps {
                script {
                    config = readYaml file: '.pipeline.yml'
                    echo "Kind: ${config.kind}"
                    echo "Job: ${config.spec.jobName}"
                    echo "Source: ${config.spec.source.repoUrl}"
                }
            }
        }

        stage('Checkout') {
            when {
                expression { config.stages.find { it.name == 'Checkout' }?.enabled == true }
            }
            steps {
                dir('sample-stig-package') {
                    git branch: "${config.spec.source.branch}",
                        url: "${config.spec.source.repoUrl}"
                }
            }
        }

        stage('Install Dependencies') {
            when {
                expression { config.stages.find { it.name == 'InstallDependencies' }?.enabled == true }
            }
            steps {
                dir('sample-stig-package') {
                    sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                    pip install -e .
                    '''
                }
            }
        }

        stage('Unit Tests') {
            when {
                expression { config.stages.find { it.name == 'UnitTests' }?.enabled == true }
            }
            steps {
                dir('sample-stig-package') {
                    sh '''
                    . venv/bin/activate
                    pytest
                    '''
                }
            }
        }

        stage('Build Package') {
            when {
                expression { config.stages.find { it.name == 'BuildPackage' }?.enabled == true }
            }
            steps {
                dir('sample-stig-package') {
                    sh '''
                    . venv/bin/activate
                    python -m build
                    '''
                }
            }
        }

        stage('Sign Package') {
            when {
                expression { config.stages.find { it.name == 'SignPackage' }?.enabled == true }
            }
            steps {
                dir('sample-stig-package') {
                    sh '''
                    . venv/bin/activate
                    python -c "from stig_package.signer import sign_file; sign_file('dist/sample_stig_package-1.0.0.tar.gz')"
                    '''
                }
            }
        }

        stage('Generate Report') {
            when {
                expression { config.stages.find { it.name == 'GenerateReport' }?.enabled == true }
            }
            steps {
                dir('sample-stig-package') {
                    sh '''
                    . venv/bin/activate
                    python -c "from stig_package.reporter import generate_report; generate_report()"
                    '''
                }
            }
        }

        stage('Publish Artifact') {
            when {
                expression { config.stages.find { it.name == 'PublishArtifact' }?.enabled == true }
            }
            steps {
                dir('sample-stig-package') {
                    sh '''
                    mkdir -p ../local-artifactory/sample-stig/releases/1.0.0
                    cp dist/*.tar.gz ../local-artifactory/sample-stig/releases/1.0.0/
                    cp dist/*.sig ../local-artifactory/sample-stig/releases/1.0.0/
                    cp reports/final_build_report.txt ../local-artifactory/sample-stig/releases/1.0.0/
                    '''
                }
            }
        }

        stage('Archive Artifacts') {
            when {
                expression { config.stages.find { it.name == 'ArchiveArtifacts' }?.enabled == true }
            }
            steps {
                archiveArtifacts artifacts: 'sample-stig-package/dist/**, sample-stig-package/reports/**', fingerprint: true
            }
        }
    }
}