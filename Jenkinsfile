def config
def isStageEnabled(stageName) {
    return config.stages.find { it.name == stageName }?.enabled == true
}

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
            when { expression { isStageEnabled('Checkout') } }
            steps {
                dir('sample-stig-package') {
                    git branch: "${config.spec.source.branch}",
                        url: "${config.spec.source.repoUrl}"
                }
            }
        }

        stage('Install Dependencies') {
            when { expression { isStageEnabled('InstallDependencies') } }
            steps {
                dir('sample-stig-package') {
                    bat '''
                    python -m venv venv
                    call venv\\Scripts\\activate
                    pip install -r requirements.txt
                    pip install -e .
                    '''
                }
            }
        }

        stage('Unit Tests') {
            when { expression { isStageEnabled('UnitTests') } }
            steps {
                dir('sample-stig-package') {
                    bat '''
                    call venv\\Scripts\\activate
                    pytest
                    '''
                }
            }
        }

        stage('Build Package') {
            when { expression { isStageEnabled('BuildPackage') } }
            steps {
                dir('sample-stig-package') {
                    bat '''
                    call venv\\Scripts\\activate
                    python -m build
                    '''
                }
            }
        }

        stage('Sign Package') {
            when { expression { isStageEnabled('SignPackage') } }
            steps {
                dir('sample-stig-package') {
                    bat '''
                    call venv\\Scripts\\activate
                    python -c "from stig_package.signer import sign_file; sign_file('dist/sample_stig_package-1.0.0.tar.gz')"
                    '''
                }
            }
        }

        stage('Generate Report') {
            when { expression { isStageEnabled('GenerateReport') } }
            steps {
                dir('sample-stig-package') {
                    bat '''
                    call venv\\Scripts\\activate
                    python -c "from stig_package.reporter import generate_report; generate_report()"
                    '''
                }
            }
        }

        stage('Publish Artifact') {
            when { expression { isStageEnabled('PublishArtifact') } }
            steps {
                dir('sample-stig-package') {
                    bat '''
                    if not exist ..\\local-artifactory\\sample-stig\\releases\\1.0.0 mkdir ..\\local-artifactory\\sample-stig\\releases\\1.0.0
                    copy dist\\*.tar.gz ..\\local-artifactory\\sample-stig\\releases\\1.0.0\\
                    copy dist\\*.sig ..\\local-artifactory\\sample-stig\\releases\\1.0.0\\
                    copy reports\\final_build_report.txt ..\\local-artifactory\\sample-stig\\releases\\1.0.0\\
                    '''
                }
            }
        }

        stage('Archive Artifacts') {
            when { expression { isStageEnabled('ArchiveArtifacts') } }
            steps {
                archiveArtifacts artifacts: 'sample-stig-package/dist/**, sample-stig-package/reports/**', fingerprint: true
            }
        }
    }
}