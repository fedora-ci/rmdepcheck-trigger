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
                       // Get every update except Flatpack and Container
                       // See the `ID Prefix`` of the releases in https://bodhi.fedoraproject.org/releases
                       [field: '$.update.release.id_prefix', expectedValue: '^(FEDORA|FEDORA-EPEL|FEDORA-EPEL-NEXT)$'],
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
                        def artifactIds = allTaskIds.collect{ "koji-build:${it}" }.join(',')

                        def branch = msg['update']['release']['branch']

                        build(
                            job: 'fedora-ci/rmdepcheck-pipeline/main',
                            wait: false,
                            parameters: [
                                string(name: 'BODHI_UPDATE_ID', value: bodhiId),
                                string(name: 'ARTIFACT_IDS', value: artifactIds),
                                string(name: 'DIST_GIT_BRANCH', value: branch),
                            ]
                        )
                    }
                }
            }
        }
    }
}
