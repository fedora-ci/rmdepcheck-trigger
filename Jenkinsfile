#!groovy


def msg
def artifactId
def releaseId
def additionalArtifactIds
def allTaskIds = [] as Set

pipeline {

    agent none

    options {
        buildDiscarder(logRotator(daysToKeepStr: '3', artifactNumToKeepStr: '100'))
        skipDefaultCheckout()
    }

    stages {
        stage('Trigger Testing') {
            steps {
                echo "Disabled for now"
            }
        }
    }
}
