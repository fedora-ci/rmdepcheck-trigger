#!groovy


def msg
def bodhiId
def allTaskIds = [] as Set

pipeline {

    agent none

    options {
        buildDiscarder(logRotator(daysToKeepStr: '3', artifactNumToKeepStr: '100'))
        skipDefaultCheckout()
    }

    triggers {
       ciBuildTrigger(
           noSquash: true,
           providerList: [
               rabbitMQSubscriber(
                   name: 'RabbitMQ',
                   overrides: [
                       topic: 'org.fedoraproject.prod.bodhi.update.status.testing.koji-build-group.build.complete',
                       queue: 'osci-pipelines-queue-rmdepcheck'
                   ],
                   checks: [
                       [field: '$.update.release.dist_tag', expectedValue: '^(f[3-9]{1}[0-9]{1})$'],
                       [field: '$.update.release.branch', expectedValue: '^(f[3-9]{1}[0-9]{1}|rawhide)$']
                   ]
               )
           ]
       )
    }

    parameters {
        string(name: 'CI_MESSAGE', defaultValue: '{}', description: 'CI Message')
    }

    stages {
        stage('Trigger Testing') {
            steps {
                script {
                    msg = readJSON text: CI_MESSAGE

                    if (msg) {

                        bodhiId = msg['update']['updateid']

                        msg['artifact']['builds'].each { build ->
                            allTaskIds.add(build['task_id'])
                        }
                        def artifactIds = allTaskIds.findAll{ it != taskId }.collect{ "koji-build:${it}" }.join(',')

                        def testProfile
                        testProfile = msg['update']['release']['dist_tag']

                        build(
                            job: 'fedora-ci/rmdepcheck-pipeline/master',
                            wait: false,
                            parameters: [
                                string(name: 'BODHI_UPDATE_ID', value: bodhiId),
                                string(name: 'ARTIFACT_IDS', value: artifactIds),
                                string(name: 'TEST_PROFILE',value: testProfile)
                            ]
                        )
                    }
                }
            }
        }
    }
}
