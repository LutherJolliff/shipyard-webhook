def shipyardBuildBadge = addEmbeddableBadgeConfiguration(id: "shipyard-build", subject: "Shipyard Build")

pipeline {
    agent {
        node {
            label 'master'
        }
    }

    environment {
        EMAIL_RECIPIANTS = 'ljolliff@cynerge.com'
        NEXUS_USER = credentials('nexus-user')
        NEXUS_PASS = credentials('nexus-pass')
        NEXUS_REPO = credentials('nexus-raw-repo')
        APP_SOURCE = './src/**/**/**/**.html'
        STATUS_SUCCESS = ''
        JOB_NAME = "${JOB_NAME}"
    }

    stages {

        stage('Dependencies') {
            agent {
                docker {
                    image 'luther007/cynerge_images'
                    args '-u root'
                    alwaysPull true
                }
            }
            steps {
                echo 'Installing...'
                sh 'echo $GIT_BRANCH'
                sh 'npm ci'
            }
        }

        // Run accessability scanning with Pa11y
        stage('Pa11y') {
            agent {
                docker {
                    image 'luther007/cynerge_images'
                    args '-u root'
                    alwaysPull true
                }
            }

            steps {
                sh 'rm -rf test-results || echo "directory does not exist"'
                sh 'mkdir test-results'
                sh 'chmod -R 777 test-results/'
                sh "pa11y-ci -T 5 ${env.APP_SOURCE} --json > test-results/pa11y-ci-results.json"
                dir('test-results') {
                    sh 'pa11y-ci-reporter-html'
                }
            }
        }

        // Run accessability scanning with Lighthouse
        stage('Lighthouse') {
            agent {
                docker {
                    image 'luther007/cynerge_images'
                    args '-u root'
                    alwaysPull true
                }
            }
            steps {
                sh 'rm -rf report/lighthouse || echo "directory does not exist"'
                sh 'mkdir -p report/lighthouse'
                sh 'chmod -R 777 report/'
                sh 'lighthouse-batch -v -h -f ./sites/sites.txt'
            }
        }
    }

    post {
        success {
            script {
                env.STATUS_SUCCESS = 'Job Complete!'
                env.JENKINS_URL = "${JENKINS_URL}"
                env.JOB_NAME = "${JOB_NAME}"

            sh 'printenv'

            // emailext body: '''${SCRIPT, template="groovy-html.template"}''',
            emailext body: '''${SCRIPT, template="email_report.template"}''',
            mimeType: 'text/html',
            subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!',
            to: "${EMAIL_RECIPIANTS}"

            }
        }

    cleanup {
        script {
            shipyardBuildBadge.setStatus('running')
            try {
                    shipyardBuildBadge.setStatus('passing')
                } catch (Exception err) {
                    shipyardBuildBadge.setStatus('failing')
                    shipyardBuildBadge.setColor('red')

                    error 'Build failed'
                }

            cleanWs()
        }
    }
    }
}
