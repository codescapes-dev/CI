
pipeline {
    agent any
    environment {
        BRANCH_NAME = "development"
        ENV = "LOWER"
        COMMIT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        TIME_STAMP = sh(returnStdout: true, script: 'date +%Y.%m.%d-%H.%M.%S').trim()
        DOCKER_TAG = ""
        DEFAULT_VERSION = 'Release-V1.0.0'
        NEW_TAG = ""
        OLD_TAG = ""
    }

    stages {

        stage('Pull') {
            steps {
               git branch: "${BRANCH_NAME}", url: 'https://github.com/codescapes-dev/CI.git'
            }
        }

        stage('Create PR') {
            steps {
                script {
                    sh "git prune"
                    sh "git fetch"
                    echo "Searching and Closing All Pull Manual PRs opened on Target Branch"
                    sh "gh pr list -s all --base main | awk '{print $1}' | xargs -I{} gh pr close {}"
                    sh "gh pr create --fill --base main"
                    sleep time: 5, unit: 'SECONDS'
                    sh "gh pr merge --merge --auto"
                }
            }
        }

        stage('Fetch Latest Release Tag') {
            steps {
                script {
                    try {
                        def latestTagOutput = sh(script: 'gh release view --json tagName', returnStdout: true).trim()
                        def json = readJSON text: latestTagOutput
                        OLD_TAG = json.tagName ?: DEFAULT_VERSION
                        NEW_TAG = bumpVersion(OLD_TAG)
                    } catch (Exception e) {
                        echo "No Prior Releases Found, Creating Default Release"
                        NEW_TAG = DEFAULT_VERSION
                    } finally {
                        echo "Latest Release Tag Is: ${NEW_TAG}"
                    }
                }
            }
        }

        stage('Create GH Release') {
            steps {
                script {
                    try {
                        echo "New: ${NEW_TAG}"
                        if (NEW_TAG == 'Release-V1.0.0') {
                            echo "If Case"
                            sh "gh release create ${NEW_TAG} --latest=true --generate-notes --target main"
                        } else {
                            echo "Else Case"
                            sh "gh release create ${NEW_TAG} --target main --notes-start-tag ${OLD_TAG} --generate-notes"
                        }
                    } catch (Exception e) {
                        echo "Error creating latest release tag: ${e.message}"
                    }
                }
            }
        }
    }
}

def bumpVersion(tag) {
    // Example: If tag is "Release-V1.1.48", extract the numeric part and increment it
    def matcher = tag =~ /(\d+)$/
    def versionNumber = matcher ? Integer.parseInt(matcher[0][1]) : 0
    def newVersion = tag.replaceFirst(/\d+$/, "${versionNumber + 1}")
    return newVersion
}
